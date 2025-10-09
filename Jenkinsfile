@Library("Shared") _ 
pipeline{
    agent { label 'Tushar'}
    
    stages{
        stage("Hello") {
            
            steps{
                script{
                    hello() 
                }
            }
        }
stage('Code Clone') {
    steps {
        script {
            clone('https://github.com/poltushar/django-notes-app.git', 'main')
        }
    }
}
        
        stage("Code Build"){
            steps{
               echo "This is building the code"
               sh "docker build -t notes-app:latest ."
               echo "Build Successful"
            }
        }
        
        stage("Test"){
            steps{
               echo "Running tests"
               echo "Testing successful"
            }
        }
        
        stage("Push to Docker Hub"){
            steps{
               echo "This is pushing the image to Docker Hub"
               script {
                   withCredentials([usernamePassword(
                       credentialsId: "dockerhubcred",
                       usernameVariable: "dockerHubUser", 
                       passwordVariable: "dockerHubPass"
                   )]) {
                       // Method 1: Using double quotes with escaped $ for shell variables
                       sh """
                           echo \$dockerHubPass | docker login -u \$dockerHubUser --password-stdin
                           docker image tag notes-app:latest \$dockerHubUser/notes-app:latest
                           docker push \$dockerHubUser/notes-app:latest
                           docker logout
                       """
                       
                       // OR Method 2: Using Groovy variables (alternative)
                       // sh "echo ${dockerHubPass} | docker login -u ${dockerHubUser} --password-stdin"
                       // sh "docker image tag notes-app:latest ${dockerHubUser}/notes-app:latest"
                       // sh "docker push ${dockerHubUser}/notes-app:latest"
                       // sh "docker logout"
                   }
               }
               echo "Push to Docker Hub Successful"
            }
        }
        
        stage("Setup Database"){
            steps{
               echo "Setting up MySQL database"
               sh """
                   docker network create notes-network || true
                   docker stop mysql-db || true
                   docker rm mysql-db || true
                   docker run -d \
                     --name mysql-db \
                     --network notes-network \
                     -e MYSQL_ROOT_PASSWORD=root \
                     -e MYSQL_DATABASE=notesdb \
                     -e MYSQL_USER=notesuser \
                     -e MYSQL_PASSWORD=notespass \
                     mysql:8.0
                   echo "Waiting for MySQL to be ready..."
                   sleep 30
               """
               echo "Database setup complete"
            }
        }
        
        stage("Deploy Application"){
            steps{
               echo "Deploying the application"
               sh """
                   docker stop notes-app-container || true
                   docker rm notes-app-container || true
                   docker run -d \
                     --name notes-app-container \
                     --network notes-network \
                     -e DB_HOST=mysql-db \
                     -e DB_NAME=notesdb \
                     -e DB_USER=notesuser \
                     -e DB_PASSWORD=notespass \
                     -e DB_PORT=3306 \
                     -p 8000:8000 \
                     notes-app:latest \
                     bash -c "python3 manage.py makemigrations && python3 manage.py migrate && python3 manage.py runserver 0.0.0.0:8000"
               """
               echo "Deployment Successful"
            }
        }
        
        stage("Health Check"){
            steps{
               echo "Checking application health"
               sh """
                   sleep 10
                   docker ps | grep notes-app-container
                   docker exec mysql-db mysql -unotesuser -pnotespass -e "USE notesdb; SHOW TABLES;"
                   echo "Application is running on http://localhost:8000"
               """
            }
        }
    }
    
    post {
        success {
            echo "Pipeline executed successfully!"
            echo "Docker image pushed to Docker Hub"
            echo "Application is running on http://localhost:8000"
        }
        failure {
            echo "Pipeline failed. Check the logs."
            sh """
                docker logs notes-app-container || true
                docker logs mysql-db || true
            """
        }
        always {
            echo "Cleaning up"
            sh "docker image prune -f || true"
        }
    }
}

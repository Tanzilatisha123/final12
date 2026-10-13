pipeline {
    agent any

    stages {

        stage('Run Selenium Tests with pytest') {
            steps {
                echo "Running Selenium Tests using pytest"

                // Install Python dependencies
                bat '"C:\\Users\\Tanzila\\AppData\\Local\\Programs\\Python\\Python310\\python.exe" -m pip install -r requirements.txt'

                // Start Flask application in background
                bat 'start "" /B "C:\\Users\\Tanzila\\AppData\\Local\\Programs\\Python\\Python310\\python.exe" app.py'

                // Wait for Flask application
                bat 'timeout /t 5 /nobreak > nul'

                // Run Selenium tests
                bat '"C:\\Users\\Tanzila\\AppData\\Local\\Programs\\Python\\Python310\\python.exe" -m pytest -v'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Build Docker Image"

                bat "docker build -t week12app:v1 ."
            }
        }

        stage('Docker Login') {
            steps {
                bat 'docker login -u tanzila123 -p Tan@1234'
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                echo "Push Docker Image to Docker Hub"

                bat "docker tag week12app:v1 tanzila123/basickubernetes:new"

                bat "docker push tanzila123/basickubernetes:new"
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo "Deploying application to Kubernetes"

                bat 'kubectl apply -f deployment.yaml --validate=false'

                bat 'kubectl apply -f service.yaml'
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }

        failure {
            echo 'Pipeline failed. Please check the logs.'
        }
    }
}
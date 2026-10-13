pipeline {
    agent any

    environment {
        PYTHON = 'C:\\Users\\Tanzila\\AppData\\Local\\Programs\\Python\\Python310\\python.exe'
        DOCKER_IMAGE = 'week12app:v1'
        DOCKER_HUB_IMAGE = 'tanzila123/basickubernetes:new'
    }

    stages {

        stage('Run Selenium Tests with pytest') {
            steps {
                echo "Running Selenium Tests using pytest"

                // Install Python dependencies
                bat "\"${PYTHON}\" -m pip install -r requirements.txt"

                // Start Flask application in background (detached)
                bat "start \"flask-app\" /B \"${PYTHON}\" app.py"

                // Wait for Flask to be ready (ping is reliable on Jenkins Windows agents)
                bat 'ping 127.0.0.1 -n 6 > nul'

                // Run Selenium tests
                bat "\"${PYTHON}\" -m pytest -v"
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Build Docker Image"
                bat "docker build -t ${DOCKER_IMAGE} ."
            }
        }

        stage('Docker Login') {
            steps {
                // Uses Jenkins Credentials (id: dockerhub-creds)
                // Create this in: Manage Jenkins > Credentials > Add Credentials
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    bat "docker login -u %DOCKER_USER% -p %DOCKER_PASS%"
                }
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                echo "Push Docker Image to Docker Hub"
                bat "docker tag ${DOCKER_IMAGE} ${DOCKER_HUB_IMAGE}"
                bat "docker push ${DOCKER_HUB_IMAGE}"
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
        always {
            echo "Cleaning up background processes..."
            // Kill the Flask app started earlier so port 5000 is free for next build
            bat 'taskkill /F /IM python.exe /FI "WINDOWTITLE eq flask-app*" || exit 0'
        }

        success {
            echo 'Pipeline completed successfully!'
        }

        failure {
            echo 'Pipeline failed. Please check the logs.'
        }
    }
}
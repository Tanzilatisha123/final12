pipeline {
    agent any

    environment {
        // ⚠️ THE KEY FIX: force Windows system dirs into PATH for every bat step
        PATH = "C:\\Windows\\System32;C:\\Windows;C:\\Windows\\System32\\Wbem;C:\\Windows\\System32\\WindowsPowerShell\\v1.0;${env.PATH}"

        PYTHON     = 'C:\\Users\\Tanzila\\AppData\\Local\\Programs\\Python\\Python310\\python.exe'
        DOCKER_IMG = 'week12app:v1'
        DOCKER_HUB = 'tanzila123/basickubernetes:new'
    }

    stages {

        stage('Run Selenium Tests with pytest') {
            steps {
                echo "Running Selenium Tests using pytest"

                // Install Python dependencies
                bat "\"${PYTHON}\" -m pip install -r requirements.txt"

                // Start Flask in background
                bat "start \"flask-app\" /B \"${PYTHON}\" app.py"

                // Wait 5 seconds for Flask to come up
                bat 'ping 127.0.0.1 -n 6 > nul'

                // Run Selenium tests
                bat "\"${PYTHON}\" -m pytest -v"
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Build Docker Image"
                bat "docker build -t ${DOCKER_IMG} ."
            }
        }

        stage('Docker Login') {
            steps {
                // Hardcoded for simplicity (as you asked). Move to Jenkins Credentials later.
                bat 'docker login -u tanzila123 -p Tan@1234'
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                echo "Push Docker Image to Docker Hub"
                bat "docker tag ${DOCKER_IMG} ${DOCKER_HUB}"
                bat "docker push ${DOCKER_HUB}"
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
            echo "Cleaning up Flask process..."
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
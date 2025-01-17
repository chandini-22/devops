pipeline {
    agent any

    environment {
        APP_NAME = "flask-app"
        DOCKER_IMAGE = "flask-app-image"
        DOCKER_PORT = "5000"
        EC2_USER = "ec2-user"
        EC2_HOST = "<YOUR_EC2_PUBLIC_IP>"
        EC2_KEY_PATH = "/path/to/your/private/key.pem"  // Update with your private key path
        VENV_DIR = "venv" // Directory for virtual environment
    }

    stages {
        stage('Clone Repository') {
            steps {
                echo "Cloning repository..."
                checkout scm
            }
        }

        stage('Set up Python Environment') {
            steps {
                echo "Setting up Python virtual environment and installing dependencies..."
                sh """
                python3 -m venv ${VENV_DIR}
                source ${VENV_DIR}/bin/activate
                pip install --upgrade pip
                pip install -r requirements.txt
                deactivate
                """
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Building Docker image..."
                sh "docker build -t ${DOCKER_IMAGE} ."
            }
        }

        stage('Push Docker Image (Optional)') {
            when {
                expression { return false } // Skip if not pushing to a Docker registry
            }
            steps {
                echo "Pushing Docker image to a registry..."
                sh "docker tag ${DOCKER_IMAGE} <YOUR_DOCKER_REGISTRY>/${DOCKER_IMAGE}"
                sh "docker push <YOUR_DOCKER_REGISTRY>/${DOCKER_IMAGE}"
            }
        }

        stage('Deploy to EC2') {
            steps {
                echo "Deploying to EC2 instance..."
                sh """
                ssh -i ${EC2_KEY_PATH} ${EC2_USER}@${EC2_HOST} << 'EOF'
                # Stop and remove any existing container
                docker stop ${APP_NAME} || true
                docker rm ${APP_NAME} || true
               
                # Pull the latest Docker image (if pushing to a registry)
                # docker pull <YOUR_DOCKER_REGISTRY>/${DOCKER_IMAGE}

                # Run the container
                docker run -d --name ${APP_NAME} -p ${DOCKER_PORT}:${DOCKER_PORT} ${DOCKER_IMAGE}
                EOF
                """
            }
        }
    }

    post {
        always {
            echo "Pipeline completed."
        }
        success {
            echo "Deployment successful!"
        }
        failure {
            echo "Deployment failed."
        }
    }
}

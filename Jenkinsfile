pipeline {
    agent any

    environment {
        IMAGE_NAME = "aceest-fitness"
        IMAGE_TAG  = "${env.BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
                echo "Source code checked out from GitHub"
            }
        }

        stage('Build Environment') {
            steps {
                sh 'pip install -r requirements.txt'
                echo "Python dependencies installed"
            }
        }

        stage('Lint') {
            steps {
                sh 'flake8 app.py test_app.py --max-line-length=100'
                echo "Lint passed"
            }
        }

        stage('Unit Tests') {
            steps {
                sh 'pytest test_app.py -v --tb=short'
            }
            post {
                always {
                    echo "Test stage complete"
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
                echo "Docker image built: ${IMAGE_NAME}:${IMAGE_TAG}"
            }
        }

        stage('Quality Gate') {
            steps {
                sh """
                    docker run --rm ${IMAGE_NAME}:${IMAGE_TAG} \
                        python -m pytest test_app.py -v --tb=short
                """
                echo "Containerized tests passed — Quality Gate cleared"
            }
        }
    }

    post {
        success {
            echo "Pipeline SUCCESS — Build #${env.BUILD_NUMBER} passed all stages"
        }
        failure {
            echo "Pipeline FAILED — Review logs for Build #${env.BUILD_NUMBER}"
        }
        always {
            sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG} || true"
        }
    }
}
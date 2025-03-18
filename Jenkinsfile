pipeline {
    agent any

    environment {
        REGISTRY = 'anuragkhapke2402'
        IMAGE_NAME = 'todo-app'
    }

    stages {
        stage('Checkout Source Code') {
            steps {
                echo 'Checking out code from Git repository...'
                checkout scmGit(
                    branches: [[name: '*/main']],
                    extensions: [],
                    userRemoteConfigs: [[url: 'https://github.com/AnuragKhapke2401/Simple-Nodejs-ToDo-App.git']]
                )
            }
        }

        stage('Set Dynamic Docker Tag') {
            steps {
                script {
                    def commit = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
                    def build = env.BUILD_NUMBER
                    env.IMAGE_TAG = "${commit}-${build}"
                    env.FULL_IMAGE = "${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}"
                    echo "Docker Image Tag: ${env.IMAGE_TAG}"
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'
                sh 'docker build -t $FULL_IMAGE .'
            }
        }

        stage('Push Docker Image to Registry') {
            steps {
                echo 'Pushing Docker image to Docker Hub...'
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push $FULL_IMAGE
                    '''
                }
            }
        }

        stage('Deploy with Docker Compose') {
            steps {
                echo 'Deploying application using Docker Compose...'
                sh '''
                    docker-compose down
                    sed -i "s|image: .*|image: $FULL_IMAGE|g" docker-compose.yaml
                    docker-compose up -d
                '''
            }
        }
    }
}

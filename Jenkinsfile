pipeline {
    agent any

    tools {
        maven 'Maven3'      
        jdk 'Java21'        
    }

    environment {
        IMAGE_NAME = "java-app"
        DOCKER_HUB_USER = "ahmedhany28" 
    }

    stages {

        stage('Check Build Number') {
            steps {
                script {
                    echo "Current Build Number: ${env.BUILD_NUMBER}"
                    if (env.BUILD_NUMBER.toInteger() < 5) {
                        error("Build number is less than 5. Failing intentionally!")
                    }
                }
            }
        }

        stage('Build with Maven') {
            steps {
                echo 'Building the Java application...'
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Docker Build') {
            steps {
                echo 'Building Docker image...'
                sh "docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} ."
            }
        }

        stage('Docker Push') {
            steps {
                echo 'Pushing Docker image to Docker Hub...'
                withCredentials([string(credentialsId: 'dockerhub-pass', variable: 'DOCKERHUB_TOKEN')]) {
                    sh """
                        echo "$DOCKERHUB_TOKEN" | docker login -u ${DOCKER_HUB_USER} --password-stdin
                        docker tag ${IMAGE_NAME}:${BUILD_NUMBER} ${DOCKER_HUB_USER}/${IMAGE_NAME}:${BUILD_NUMBER}
                        docker push ${DOCKER_HUB_USER}/${IMAGE_NAME}:${BUILD_NUMBER}
                    """
                }
            }
        }

        stage('Deploy on Docker') {
            steps {
                echo 'Deploying the container...'
                sh """
                    docker rm -f java-app || true
                    docker run -d --name java-app -p 8090:8090 ${DOCKER_HUB_USER}/${IMAGE_NAME}:${BUILD_NUMBER}
                """
            }
        }
    }

    post {
        always {
            echo 'Cleaning workspace...'
            cleanWs()
        }

        success {
            echo 'SUCCESS'
        }

        failure {
            echo 'FAILED'
        }
    }
}

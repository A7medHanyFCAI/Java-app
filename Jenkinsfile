pipeline {
    agent any

    tools {
        maven 'Maven3'      // Maven tool configured in Jenkins
        jdk 'Java21'        // JDK tool configured in Jenkins
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
            when {
                expression { env.BUILD_NUMBER.toInteger() >= 5 }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo "Logging into Docker Hub..."
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        echo "Tagging image..."
                        docker tag ${IMAGE_NAME}:${BUILD_NUMBER} $DOCKER_USER/${IMAGE_NAME}:${BUILD_NUMBER}
                        echo "Pushing image to Docker Hub..."
                        docker push $DOCKER_USER/${IMAGE_NAME}:${BUILD_NUMBER}
                        docker logout
                    '''
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
            echo 'Build SUCCESS!'
        }

        failure {
            echo 'Build FAILED.'
        }
    }
}

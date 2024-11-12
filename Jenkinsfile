pipeline {
    agent any

    tools {
        jdk 'jdk17'
        maven 'maven3'
    }

    environment {
        SONAR_HOST_URL = 'http://localhost:9000'
        SONAR_TOKEN = 'squ_07f2dc1e2bb071dd862d9337e36eb01814c4d722'
        DOCKER_IMAGE_NAME = 'spring-boot-prof-management'
    }

    stages {
        stage('Code Checkout') {
            steps {
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/AbderrahmaneOd/Spring-Boot-Jenkins-CI-CD'
            }
        }
          stage('Clean & Package') {
            steps {
                bat "mvn clean package -DskipTests"
            }
        }
stage('Docker Build & Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'DockerHub-Token', toolName: 'docker') {
                        def buildTag = "${DOCKER_IMAGE_NAME}:${BUILD_NUMBER}"
                        def latestTag = "${DOCKER_IMAGE_NAME}:latest"
                        
                        bat "docker build -t ${DOCKER_IMAGE_NAME} -f Dockerfile.final ."
                        bat "docker tag ${DOCKER_IMAGE_NAME} abdeod/${buildTag}"
                        bat "docker tag ${DOCKER_IMAGE_NAME} abdeod/${latestTag}"
                        bat "docker push abdeod/${buildTag}"
                        bat "docker push abdeod/${latestTag}"
                        env.BUILD_TAG = buildTag
                    }
                }
            }
        }
        stage('SonarQube Analysis') {
            steps {
                bat """
                mvn clean sonar:sonar ^
                  -Dsonar.host.url=${SONAR_HOST_URL} ^
                  -Dsonar.login=${SONAR_TOKEN}
                """
            }
        }

      

        

        stage('Vulnerability Scanning') {
            steps {
                bat "trivy image abdeod/${env.BUILD_TAG}"
            }
        }

        stage('Staging') {
            steps {
                bat 'docker-compose up -d'
            }
        }
    }
}

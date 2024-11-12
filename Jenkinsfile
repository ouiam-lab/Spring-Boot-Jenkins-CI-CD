pipeline {
    agent any

    tools {
        jdk 'jdk17'
        maven 'maven3'
    }

    environment {
        SONAR_HOST_URL = 'http://localhost:9000' // Update if SonarQube is hosted elsewhere
        SONAR_TOKEN = 'squ_07f2dc1e2bb071dd862d9337e36eb01814c4d722' // Directly setting the token here
    }

    stages {
        stage('Code Checkout') {
            steps {
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/AbderrahmaneOd/Spring-Boot-Jenkins-CI-CD'
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

        stage('Clean & Package') {
            steps {
                bat "mvn clean package -DskipTests"
            }
        }

        stage("Docker Build & Push") {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'DockerHub-Token', toolName: 'docker') {
                        def imageName = "spring-boot-prof-management"
                        def buildTag = "${imageName}:${BUILD_NUMBER}"
                        def latestTag = "${imageName}:latest"

                        bat "docker build -t ${imageName} -f Dockerfile.final ."
                        bat "docker tag ${imageName} abdeod/${buildTag}"
                        bat "docker tag ${imageName} abdeod/${latestTag}"
                        bat "docker push abdeod/${buildTag}"
                        bat "docker push abdeod/${latestTag}"
                        env.BUILD_TAG = buildTag
                    }
                }
            }
        }

        stage('Vulnerability Scanning') {
            steps {
                bat "trivy image abdeod/${BUILD_TAG}"
            }
        }

        stage("Staging") {
            steps {
                bat 'docker-compose up -d'
            }
        }
    }
}

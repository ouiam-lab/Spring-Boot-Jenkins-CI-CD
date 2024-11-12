pipeline {
    agent any

    tools{
        jdk 'jdk17'
        maven 'maven3'
    }

    stages {
        stage('Code Checkout') {
            steps {
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/AbderrahmaneOd/Spring-Boot-Jenkins-CI-CD'
            }
        }
        
        

        stage('Sonarqube Analysis') {
            steps {
                bat ''' mvn sonar:sonar \
                    -Dsonar.host.url=http://localhost:9000/ \
                    mvn sonar:sonar -Dsonar.host.url=http://127.0.0.1:9000 -Dsonar.login=squ_b19338fc39680b9c356a743decb50627e2b401a7
 '''
            }
        }

        stage('Clean & Package'){
            steps{
                bat "mvn clean package -DskipTests"
            }
        }


        
       stage("Docker Build & Push"){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'DockerHub-Token', toolName: 'docker') {
                        def imageName = "spring-boot-prof-management"
                        def buildTag = "${imageName}:${BUILD_NUMBER}"
                        def latestTag = "${imageName}:latest"  // Define latest tag
                        
                        bat "docker build -t ${imageName} -f Dockerfile.final ."
                        bat "docker tag ${imageName} abdeod/${buildTag}"
                        bat "docker tag ${imageName} abdeod/${latestTag}"  // Tag with latest
                        bat "docker push abdeod/${buildTag}"
                        bat "docker push abdeod/${latestTag}"  // Push latest tag
                        env.BUILD_TAG = buildTag
                    }
                        
                }
            }
        }
        
        stage('Vulnerability scanning'){
            steps{
                bat " trivy image abdeod/${buildTag}"
            }
        }

        stage("Staging"){
            steps{
                bat 'docker-compose up -d'
            }
        }
    }
}

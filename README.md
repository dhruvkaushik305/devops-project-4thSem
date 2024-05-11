# DevOps Project
### By Dhruv Kaushik (E22CSEU1638)
This repo contains the demo project that was used in the jenkins pipeline
Additionally the tools used are:
- Jenkins
- Docker
- SonarQube
- OWASP
- Git

The jenkins pipeline is as follows
```
pipeline {
    agent any
    tools{
        jdk 'jdk-17'
        maven 'maven3.9.6'
    }
    environment{
        SCANNER_HOME= tool 'sonar'
    }
    stages {
        stage('Git checkout') {
            steps {
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/jaiswaladi246/java-docker-genie.git'
            }
        }
        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }
        stage('Sonarqube analysis') {
            steps {
                sh  "${env.SCANNER_HOME}/bin/sonar-scanner -Dsonar.url=http://localhost:9000 -Dsonar.login=squ_29a43be8598ee0149af9d089b31aaf27a0a22437 -Dsonar.projectName=docker-desktop -Dsonar.java.binaries=. -Dsonar.projectKey=docker-desktop"
            }
        }
         stage('OWASP') {
            steps {
               dependencyCheck additionalArguments: ' --scan ./ --format HTML', odcInstallation: 'owasp'
               dependencyCheckPublisher pattern: "**/dependency-check-report.html"
            }
        }
        stage('Build') {
            steps {
                sh "mvn clean install"
            }
        }
        stage('Docker build and Push') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'aeaf7d18-847a-4786-b137-16be94bc6530') {
                        sh "docker build -t docker-desktop ."
                        sh "docker tag docker-desktop dhruvkaushik/docker-desktop:latest"
                        sh "docker push dhruvkaushik/docker-desktop:latest"
                    }
                }
            }
        }
        stage('Docker deploy') {
            steps {
                script{
                    def existingContainerId = sh(script: "docker ps -q -f name=docker-desktop", returnStdout: true).trim()
                    if (existingContainerId) {
                        sh "docker kill docker-desktop"
                        sh "docker rm docker-desktop"
                    }
                     withDockerRegistry(credentialsId: 'aeaf7d18-847a-4786-b137-16be94bc6530'){
                         sh "docker run -d --name docker-desktop -p 8081:8080 dhruvkaushik/docker-desktop:latest"
                     }
                }
            }
        }
    }
}
```

//This file has been modified by Itohan Eregie on 29/04/2024
pipeline{
     agent{  
         label "wsl"
       
    }
    tools {
        jdk 'Java17' //this installs java 
        maven 'Maven3'
       
    }
    environment {
        APP_NAME = "production-pipeline "
        RELEASE_NUMBER = "1.0.0"
        DOCKER_USER = "sweetpeaito"
        DOCKER_PASS = "docker"//This is a secret that will be set up and used to sign into docker. it will be setup in docker hub as an access token
        // IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
        IMAGE_NAME = "sweetpeaito/production-pipeline"
        IMAGE_TAG = "${RELEASE_NUMBER}-${BUILD_NUMBER}"
        JENKINS_API_TOKEN = credentials("JENKINS_API_TOKEN")
    
    }
    stages{
        stage("Cleanup Workspace"){
            steps {
                cleanWs()
            }

        }
    
        stage("Checkout from SCM"){//check out the code from git repo into the jenkins workspace 
            steps {
                git branch: 'main', credentialsId: 'git', url: 'https://github.com/techinanutshellhack/production-devops.git'
            }

        }

        stage("Build Application"){//build application 
            steps {
               sh  "mvn --version" 
                sh "mvn clean package"
            }

        }

        stage("Test Application"){// test application with maven
            steps {
                sh "mvn test" 
            }

        }
        stage("Sonarqube Analysis") {
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token') {
                        sh "mvn sonar:sonar"
                    }
                }
            }

        }

        stage("Quality Gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'jenkins-sonarqube-token'
                }
            }

        }
        stage("Build & Push Docker Image") {
            steps {
                script {
                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image = docker.build "${IMAGE_NAME}"
                    }

                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push('latest')
                    }
                }
            }

        }
        
        stage ('Cleanup Artifacts') {
             steps {
                 script {
                     sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                     sh "docker rmi ${IMAGE_NAME}:latest"
                 }
             }
        }
        stage ('Trigger CD Pipeline'){//trigger the continuous deployment pipeline 
            steps {
                    script {
                        sh "curl -v -k --user admin:${JENKINS_API_TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' 'http://localhost:8080/job/application-deployment/buildWithParameters?token=application-deployment-token'"
                    }
                }
 
        }
        
    }
}
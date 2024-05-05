//This file has been modified by Itohan Eregie on 29/04/2024
pipeline{
     agent{  //the agent is the vm that the all the dependencies for the jenkins pipeline to run in will be installed . in production , use a dedicated agent
         //label "built-in"
         label "wsl"
        //docker { image 'node:20.10.0-alpine3.19' }
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
        IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
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
        
        stage('Docker Build and Push'){
          
          steps{
            echo 'Docker build app'
            script{
                    docker.withRegistry('',DOCKER_PASS ) {
                            docker_image = docker.build "${IMAGE_NAME}"
                            docker_image.push("${IMAGE_TAG}")
                            docker_image.push("latest")
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
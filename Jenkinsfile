
pipeline{
    agent{  //the agent is the vm that the all the dependencies for the jenkins pipeline to run in will be installed . in production , use a dedicated agent
        label "built-in"
    }
    tools {
        jdk 'Java17'
        maven 'Maven3'
    }
    environment {
        APP_NAME = "production-pipeline"
        RELEASE = "1.0.0"
        DOCKER_USER = "sweetpeaito"
        DOCKER_PASS = 'dockerhub'//this is a secret that will be set up and used to sign into docker. it wull be setup in docker hub as an access token
        IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
      //  JENKINS_API_TOKEN = credentials("JENKINS_API_TOKEN")

    }
    stages{
        stage("Cleanup Workspace"){
            steps {
                cleanWs()
            }

        }
    
        stage("Checkout from SCM"){//check out the code from git repo into the jenkins workspace 
            steps {
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/techinanutshellhack/production-devops.git'
            }

        }

        stage("Build Application"){//build application 
            steps {
                sh "mvn clean package"
            }

        }

        stage("Test Application"){// test application with maven
            steps {
                sh "mvn test"
            }

        }
        
        // stage("Sonarqube Analysis") {//send code from source code repo to sonarcube for analysis
        //     steps {
        //         script {
        //             withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token') {
        //                 sh "mvn sonar:sonar"
        //             }
        //         }
        //     }

        // }

    //    stage("Quality Gate") {
    //          steps {
    //             script {
    //                  waitForQualityGate abortPipeline: false, credentialsId: 'jenkins-sonarqube-token'
    //              }
    //          }

    //      }

        //  stage("Build & Push Docker Image") {
        //      steps {
        //          script {
        //              docker.withRegistry('https://index.docker.io/v1/',DOCKER_PASS) {
        //                  docker_image = docker.build "${IMAGE_NAME}"
        //              }

        //              docker.withRegistry('',DOCKER_PASS) {
        //                  docker_image.push("${IMAGE_TAG}")
        //                  docker_image.push('latest')
        //              }
        //          }
        //      }

        //  }
        stage('Docker Build and Push'){
          
          steps{
            echo 'Packaging demo app with docker'
            script{
              docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
                  docker_image = docker.build "${IMAGE_NAME}"
                  docker_image.push()
                  docker_image.push("dev")
              docker_image.push("latest")
              }
            }
          }
      }

    //      stage("Trivy Scan") {
    //          steps {
    //              script {
	// 	   sh ('docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image sweetpeaito/production-pipeline:1.0.0-22 --no-progress --scanners vuln  --exit-code 0 --severity HIGH,CRITICAL --format table')
    //              }
    //          }

    //     }

    //      stage ('Cleanup Artifacts') {
    //          steps {
    //              script {
    //                  sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
    //                  sh "docker rmi ${IMAGE_NAME}:latest"
    //              }
    //          }
    //      }


    //      stage("Trigger CD Pipeline") {
    //          steps {
    //              script {
    //                  sh "curl -v -k --user admin:${JENKINS_API_TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' 'https://jenkins.dev.dman.cloud/job/gitops-complete-pipeline/buildWithParameters?token=gitops-token'"
    //              }
    //          }

    //      }

    //  }

    //  post {
    //      failure {
    //      emailext body: '''${SCRIPT, template="groovy-html.template"}''', 
    //                  subject: "${env.JOB_NAME} - Build # ${env.BUILD_NUMBER} - Failed", 
    //                  mimeType: 'text/html',to: "itohaneregie@gmail.com"
    //          }
    //       success {
    //             emailext body: '''${SCRIPT, template="groovy-html.template"}''', 
    //                  subject: "${env.JOB_NAME} - Build # ${env.BUILD_NUMBER} - Successful", 
    //                  mimeType: 'text/html',to: "itohaneregie@gmail.com"
    //        }      
     }
}

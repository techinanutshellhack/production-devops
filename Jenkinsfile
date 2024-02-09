//the agent is the machine that runs the jobs and as such it should have all the dependencies it requires to execute the job . for example 
//docker . docker docker cki must be installed on the agent in order to run docker build commands 
pipeline{

     agent{  //the agent is the vm that the all the dependencies for the jenkins pipeline to run in will be installed . in production , use a dedicated agent
         //label "built-in"
         label "jenkins-agent"
        //docker { image 'node:20.10.0-alpine3.19' }
    }
    tools {
        jdk 'Java17' //this installs java to the agent. you can also install this directly on the agent
        maven 'Maven3' // this install maven to the agent
    }
    environment {
        APP_NAME = "production-pipeline"
        RELEASE = "1.0.0"
        DOCKER_USER = "sweetpeaito"
        DOCKER_PASS = 'dockerhub'//this is a secret that will be set up and used to sign into docker. it wull be setup in docker hub as an access token
        IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
       //JENKINS_API_TOKEN = credentials("JENKINS_API_TOKEN")

    }
    stages{
        stage("Cleanup Workspace"){
            steps {
                cleanWs()
            }

        }
    
        stage("Checkout from SCM"){//check out the code from git repo into the jenkins workspace 
            steps {
                git branch: 'jenkins', credentialsId: 'github', url: 'https://github.com/techinanutshellhack/production-devops'
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
        
        stage("Sonarqube Analysis") {//send code from source code repo to sonarcube for analysis
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'sonarqube-jenkins') {
                        sh "mvn sonar:sonar"
                    }
                }
            }

        }

        stage("Quality Gate") {
              steps {
                 script {
                      waitForQualityGate abortPipeline: false, credentialsId: 'sonarqube-jenkins'
                  }
              }

          }

//         //  stage("Build & Push Docker Image") {
//         //      steps {
//         //          script {
//         //              docker.withRegistry('https://index.docker.io/v1/',DOCKER_PASS) {
//         //                  docker_image = docker.build "${IMAGE_NAME}"
//         //              }

//         //              docker.withRegistry('',DOCKER_PASS) {
//         //                  docker_image.push("${IMAGE_TAG}")
//         //                  docker_image.push('latest')
//         //              }
//         //          }
//         //      }

//         //  }
//         stage('Docker Build and Push'){
          
//           steps{
//             echo 'Packaging demo app with docker'
//             script{
//                     docker.withRegistry('',DOCKER_PASS ) {
//                             docker_image = docker.build "${IMAGE_NAME}"
//                            // docker_image.push()
//                             docker_image.push("${IMAGE_TAG}")
//                             docker_image.push("latest")
//                    // dockerImage = docker.build registry + "latest"
//               }
//             }
//           }
//       }
       

//     //      stage("Trivy Scan") {
//     //          steps {
//     //              script {
// 	// 	   sh ('docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image sweetpeaito/production-pipeline:1.0.0-22 --no-progress --scanners vuln  --exit-code 0 --severity HIGH,CRITICAL --format table')
//     //              }
//     //          }

//     //     }

//          stage ('Cleanup Artifacts') {
//              steps {
//                  script {
//                      sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
//                      sh "docker rmi ${IMAGE_NAME}:latest"
//                  }
//              }
//          }


//           stage("Trigger CD Pipeline") {
//               steps {
//                   script {
//                       sh "curl -v -k --user admin:${JENKINS_API_TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' 'http://http://ec2-16-171-199-144.eu-north-1.compute.amazonaws.com:8080/:8080/job/application/buildWithParameters?token=application'"
//                   }
//               }

//         //  }

//      }

//     //   post {
//     //       failure {
//     //       emailext body: '''${SCRIPT, template="groovy-html.template"}''', 
//     //                   subject: "${env.JOB_NAME} - Build # ${env.BUILD_NUMBER} - Failed", 
//     //                   mimeType: 'text/html',to: "itohaneregie@gmail.com"
//     //           }
//     //        success {
//     //              emailext body: '''${SCRIPT, template="groovy-html.template"}''', 
//     //                   subject: "${env.JOB_NAME} - Build # ${env.BUILD_NUMBER} - Successful", 
//     //                   mimeType: 'text/html',to: "itohaneregie@gmail.com"
//     //        }      
  }
}
pipeline {
    agent any
    environment {
        IMAGE_REPO_NAME="log4shellpub"
        //REPLACE XXX WITH YOUR STUDENT NUMBER
        IMAGE_TAG= "ebcdemo"        
        REPOSITORY_URI = "public.ecr.aws/f9n2h3p5/log4shellpub"
        AWS_DEFAULT_REGION = "us-east-1"
        scannerHome = tool 'SonarScanner'
    }
   
    stages {
    
            stage('Logging into AWS ECR') {
            steps {
                script {
                sh """aws ecr-public get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${REPOSITORY_URI} """
                }
                 
            }
        } 
    
    stage('Clone repository') { 
            steps { 
                script{
                checkout scm
                }
            }
        }  
  
    // Building Docker images
    stage('Building image') {
      steps{
        script {
          dockerImage = docker.build "${IMAGE_REPO_NAME}:${IMAGE_TAG}-${env.BUILD_NUMBER}"
        }
      }
    }
   
    // Uploading Docker images into AWS ECR
    stage('Pushing to ECR') {
     steps{  
         script {
                sh """docker tag ${IMAGE_REPO_NAME}:${IMAGE_TAG}-${env.BUILD_NUMBER} ${REPOSITORY_URI}:$IMAGE_TAG-${env.BUILD_NUMBER}"""
                sh """docker push ${REPOSITORY_URI}:${IMAGE_TAG}-${env.BUILD_NUMBER}"""
         }
        }
      }
      stage('Deploy'){
            steps {
                 sh 'sed -i "s/<TAG>/${IMAGE_TAG}-${BUILD_NUMBER}/" deployment.yml'
                 sh 'kubectl apply -f deployment.yml'
                 /*
                 //If you are sure this deployment is already running and want to change the container image version, then you can use:
                 sh 'kubectl set image deployments/dvwa 371571523880.dkr.ecr.us-east-2.amazonaws.com/dvwaxperts:${BUILD_NUMBER}'*/
            }
        } 


      stage('SonarQube analysis') {
            steps {
                withSonarQubeEnv() { // If you have configured more than one global server connection, you can specify its name
                sh "${scannerHome}/bin/sonar-scanner"
                }
            }
      }




        
       /*stage('FortiDevSec DAST'){
            steps {
                 sh 'env | grep -E "JENKINS_HOME|BUILD_ID|GIT_BRANCH|GIT_COMMIT" > /tmp/env'
                 sh 'docker pull registry.fortidevsec.forticloud.com/fdevsec_dast:latest'
                 sh 'docker run --rm --env-file /tmp/env --mount type=bind,source=$PWD,target=/scan registry.fortidevsec.forticloud.com/fdevsec_dast:latest'
            }
        }*/
    }
}

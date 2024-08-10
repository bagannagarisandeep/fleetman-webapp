pipeline {
   agent any

   environment {
     // You must set the following environment variables
     // ORGANIZATION_NAME
     // YOUR_DOCKERHUB_USERNAME (it doesn't matter if you don't have one)
     
     ORGANIZATION_NAME = "bagannagarisandeep" 
     SERVICE_NAME = "fleetman-webapp"
     ECR_URI = "730335413903.dkr.ecr.ap-south-1.amazonaws.com/fleetman"
     REPOSITORY_TAG ="${ECR_URI}:${BUILD_ID}"
   }

   stages {
      stage('Preparation') {
         steps {
            cleanWs()
            git credentialsId: 'GitHub', url: "https://github.com/${ORGANIZATION_NAME}/${SERVICE_NAME}"
         }
      }
      stage('Build') {
	  tools {
              maven 'maven'
          }
         steps {
            sh "mvn -DskipTests clean package"
            archiveArtifacts artifacts: '**/target/*.war', allowEmptyArchive: true
         }
      }

      stage('Build and Push Image') {
	      steps {	
            script {  
              withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'AWS Credentials']]) {
              sh 'aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 730335413903.dkr.ecr.ap-south-1.amazonaws.com'
               }	
            }
             sh 'docker buildx build -t ${REPOSITORY_TAG} .'
             sh 'docker push ${REPOSITORY_TAG}'
         }
      }
	   
stage('Deploy to Cluster') {
     steps {
	sh 'envsubst < ${WORKSPACE}/deploy.yaml | /usr/local/bin/kubectl --kubeconfig ${WORKSPACE}/jenkins-cluster-admin-config apply -f -'
       }
	   }
	  }
    }

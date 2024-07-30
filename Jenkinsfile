pipeline {
   tools {
        maven 'Maven'
    }
    agent {label 'slave-node'}
    environment {
        registry = "799344209838.dkr.ecr.us-east-1.amazonaws.com/shank04"
    }
    
    stages {
        stage('Cloning Git') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/main']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: '', url: 'https://github.com/shashank6613/horizon.git']]])     
            }
        }
        stage ('Build') {
          steps {
            sh 'mvn clean install'           
            }
        }
        
            // Building Docker images
        stage('Building image') {
          steps{
            script {
              sh 'docker build -t $JOB_NAME:v1.$BUILD_ID .'
              sh 'docker tag $JOB_NAME:v1.$BUILD_ID ${registry}:v1.$BUILD_ID'
             }
           }
         }
   
    // Uploading Docker images into AWS ECR
       stage('Pushing to ECR') {
         steps{  
           script {
             sh 'aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 799344209838.dkr.ecr.us-east-1.amazonaws.com/shank04' 
             sh 'docker push ${registry}:v1.$BUILD_ID'
             sh 'docker rmi $JOB_NAME:v1.$BUILD_ID ${registry}:v1.$BUILD_ID' // Delete docker images from server 
           }
          }
         }
       
       stage('K8S Deploy') {
       steps{   
         script {
            withKubeConfig([credentialsId: 'anyname', serverUrl: '']) {
              sh '<curl -LO “https://storage.googleapis.com/kubernetes-release/release/v1.20.5/bin/linux/amd64/kubectl">'
              sh 'chmod u+x ./kubectl'
              sh 'envsubst < eks-deploy-k8s.yaml > eks-deploy-k8s1.yaml'
              sh './kubectl apply -f eks-deploy-k8s.yaml'
             }
           }
        }
       }
    }
}

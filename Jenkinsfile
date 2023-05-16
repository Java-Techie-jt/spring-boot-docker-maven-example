pipeline {

    agent any
   
    stages {      
        stage('Git Checkout') {
            steps { 
                    echo "Checking out code from github"
                    checkout scm
                 }
               }      
              
        stage('Build Stage') {
           agent { docker 'maven:3.5-alpine' }
           steps { 
                   echo 'Building stage for the app...'
                   sh 'mvn compile'
           }
        }

        stage('Test App') {
           agent { docker 'maven:3.5-alpine' }
           steps {
                   echo 'Testing stage for the app...'
                   sh 'mvn test'
                   junit '**/target/surefire-reports/TEST-*.xml'

           }
        }

        stage('Packaging Stage') {
           agent { docker 'maven:3.5-alpine' }
           steps {
                   echo 'Packaging stage for the app...'
                   sh 'mvn package'
           }
        }

        stage('Docker Image Build') {
            steps {
                echo 'Bulding docker image...'
                sh "docker build -t product_service:{env.BUILD_NUMBER} ."
            }
        }        

        stage('Push Docker Image to ECR') {
            steps {
                withAWS(credentials: 'AWS_CREDENTIALS_ID', region: 'us-west-1') {
                    sh 'aws ecr get-login-password --region us-west-1 | docker login --username AWS --password-stdin 634639955940.dkr.ecr.us-west-1.amazonaws.com'
                    sh 'docker tag product_service:${env.BUILD_NUMBER} 634639955940.dkr.ecr.us-west-1.amazonaws.com/product_service:${env.BUILD_NUMBER}'
                    sh 'docker push 634639955940.dkr.ecr.us-west-1.amazonaws.com/product_service:${env.BUILD_NUMBER}'
                 }
              }
           }       
        }

    post {
      failure {
        mail to: 'richgoldd2@gmail.com',
            subject: 'Failed pipeline: ${currentBuild.fullDisplayName}',
            body: 'Pipeline failed for dev ${env.BUILD_URL}'
         }
       }
     }
  

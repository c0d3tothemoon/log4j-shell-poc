pipeline {
  environment {
    dockerimagename = "archstein/log4shell"
    dockerImage = ""
  }
  agent any
  stages {
    stage('Checkout Source') {
      steps {
        git 'https://github.com/c0d3tothemoon/jenkins-kubernetes-deployment.git'
      }
    }
    stage('Build image') {
      steps{
        script {
          dockerImage = docker.build dockerimagename
        }
      }
    }
    stage('Pushing Image') {
      environment {
          registryCredential = 'archstein'
           }
      steps{
        script {
          docker.withRegistry( 'https://registry.hub.docker.com', registryCredential ) {
          dockerImage.push("latest")
          }
        }
      }
    }
    stage ('K8s Get Node') {
        steps{    
        
        withKubeCredentials(kubectlCredentials: [[caCertificate: '', clusterName: 'sample', contextName: '', credentialsId: 'Jenkins_serviceAccount', namespace: 'default', serverUrl: 'https://172.16.16.180:6443']]) 
        {
        sh './kubectl apply -f deployment.yaml'
      }
    }
   }
 }
}

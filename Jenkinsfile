pipeline {
  environment {
    dockerimagename = "archstein/log4shell"
    dockerImage = ""
  }
  agent any
  stages {
    stage('Clone repository') { 
            steps { 
                script{
                checkout scm
                }
            }
        }
      stage('Code Scanning - SAST'){
          steps {
                 sh 'env | grep -E "JENKINS_HOME|BUILD_ID|GIT_BRANCH|GIT_COMMIT" > /tmp/env'
                 sh 'docker pull registry.fortidevsec.forticloud.com/fdevsec_sast:latest'
                 sh 'docker run --rm --env-file /tmp/env --mount type=bind,source=$PWD,target=/scan registry.fortidevsec.forticloud.com/fdevsec_sast:latest'
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
    stage ('Image Scan by FortiCNP'){
      steps{
      fortiCWPScanner block: true, imageName: 'archstein/log4shell:latest'
      }
    }
    stage ('Deploy Container in Kubernetes') {
        steps{    
        
        withKubeCredentials(kubectlCredentials: [[caCertificate: '', clusterName: 'sample', contextName: '', credentialsId: 'Jenkins_serviceAccount', namespace: 'default', serverUrl: 'https://172.16.16.180:6443']]) 
        {
        sh 'kubectl apply -f deployment.yml'
      }
    }
   }
   stage('DAST by FortiDAST'){
      steps {
         sh 'env | grep -E "JENKINS_HOME|BUILD_ID|GIT_BRANCH|GIT_COMMIT" > /tmp/env'
         sh 'docker pull registry.fortidevsec.forticloud.com/fdevsec_dast:latest'
         sh 'docker run --rm --env-file /tmp/env --mount type=bind,source=$PWD,target=/scan registry.fortidevsec.forticloud.com/fdevsec_dast:latest'
            }
        }   
 }
}

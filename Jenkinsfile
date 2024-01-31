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
        sh 'export EMAIL=sleman@fortinet.com LICENSE_SERIAL=FDEVSC0000000591_8189SZ479476 ASSET_TOKEN=iqjhK7WydnRPZ0sPMCT61M43I1zel6Dtac+/QdEcf9LpaIDhMyWSNF1yRdjwwhmLk4nAxi/IJhsR7AEGXdaSUSrI+axLM1qUN0QqJQ== SCANURL=http://172.16.16.180:30080 SCANTYPE=1 ASSET=a1c92d83-e7c3-4b3d-8b6d-067d99895d5d'
        sh 'env | grep -E "EMAIL|LICENSE_SERIAL|ASSET_TOKEN|SCANURL|SCANTYPE|ASSET" > /tmp/env'
        sh 'docker pull registry.fortidast.forticloud.com/dastdevopsproxy:latest'
        sh 'docker run --rm --env-file /tmp/env --network=host registry.fortidast.forticloud.com/dastdevopsproxy:latest'
            }
        }   
 }
}

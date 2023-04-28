node{
    stage('git checkout'){
       git 'https://github.com/suthapai/kubernetesProject1.git' 
    }
    stage('sending dockerfile to ansible server over ssh'){
       sshagent(['ansible_demo']) {
              sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.93.134'
              sh 'scp /var/lib/jenkins/workspace/pipeline_demo/*  ubuntu@172.31.93.134:/home/ubuntu'
       }
    }
    stage('docker build image'){
       sshagent(['ansible_demo']) {
              sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.93.134 cd /home/ubuntu/'
              sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.93.134 docker image build -t $JOB_NAME:v1.$BUILD_ID . '
        }
    }
    stage('docker image tagging'){
       sshagent(['ansible_demo']) {
              sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.93.134 cd /home/ubuntu/'
              sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.93.134 docker image tag $JOB_NAME:v1.$BUILD_ID sujetha/$JOB_NAME:v1.$BUILD_ID '
               sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.93.134 docker image tag $JOB_NAME:v1.$BUILD_ID sujetha/$JOB_NAME:latest '
        }
    }
    stage('push docker image to dockerhub'){
       sshagent(['ansible_demo']) {
           
           withCredentials([string(credentialsId: 'dockerHubPassword', variable: 'dockerHubPassword')]) {
                 sh "ssh -o StrictHostKeyChecking=no ubuntu@172.31.93.134 docker login -u sujetha -p ${dockerHubPassword}"
                 sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.93.134 docker image push sujetha/$JOB_NAME:v1.$BUILD_ID '
                 sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.93.134 docker image push sujetha/$JOB_NAME:latest '
                 
                 }
            
        }
    }
    
    
}
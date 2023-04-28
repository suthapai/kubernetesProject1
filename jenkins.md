node{
  stage("git checkout"){
      git 'https://github.com/suthapai/kubernetesProject1.git'
  }
   stage("Sending files to ansible server over ssh"){
       
     sshagent(['demo_ansible']) { 
        sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.80.133'
        sh 'scp /var/lib/jenkins/workspace/pineline_demo1/**  ubuntu@172.31.80.133:/home/ubuntu'
        
    
     }
  }
  stage("Build Docker Image"){
       sshagent(['demo_ansible']) {
        sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.80.133 cd /home/ubuntu/'
        sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.80.133 docker image build -t $JOB_NAME:v1.$BUILD_ID .'
    
     }
  }
   stage(" Docker Image tagging"){
       sshagent(['demo_ansible']){
        sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.80.133 cd /home/ubuntu/'
        sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.80.133 docker image tag $JOB_NAME:v1.$BUILD_ID sujetha/$JOB_NAME:v1.$BUILD_ID'
    sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.80.133 docker image tag $JOB_NAME:v1.$BUILD_ID sujetha/$JOB_NAME:latest'
     }
  }
  stage(" Pushing Image to DockerHub"){
      sshagent(['demo_ansible']){
           withCredentials([string(credentialsId: 'dockerhub_password', variable: 'dockerhub_password')]) {
           sh "ssh -o StrictHostKeyChecking=no ubuntu@172.31.80.133 docker login -u sujetha -p ${dockerhub_password}"
           sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.80.133 docker image push sujetha/$JOB_NAME:v1.$BUILD_ID'
           sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.80.133 docker image push sujetha/$JOB_NAME:latest'
           sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.80.133 docker image rm sujetha/$JOB_NAME:latest sujetha/$JOB_NAME:v1.$BUILD_ID $JOB_NAME:v1.$BUILD_ID'
            }
           
        } 
     }
        stage("Copying files from ansible to kubernetes server "){
       
         sshagent(['Kubernetes_server']) {
               sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.85.116'
               sh 'scp /var/lib/jenkins/workspace/pineline_demo1/*  ubuntu@172.31.85.116:/home/ubuntu/'

            }
        }
      
       
         stage("Kubernetes Deployment Using Ansible"){
           sshagent(['ansible_demo']) {
               
           sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.80.133 cd /home/ubuntu/'
           sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.80.133 ansible-playbook ansible.yml'
  
                }
        }
  
  
  
  
}

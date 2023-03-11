pipeline {
  agent any
  tools {
  
  maven 'maven'
   
  }
    stages {

      stage ('Checkout SCM'){
        steps {
          checkout([$class: 'GitSCM', branches: [[name: '*/main']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'git', url: 'https://github.com/soma2907/Devops_Project.git']]])
        }
      }
	  
	  stage ('Build')  {
	      steps {
          
            dir('java-source'){
            sh "mvn package"
          }
        }
         
      }

    stage ('Artifactory configuration') {
            steps {
                rtServer (
                    id: "jfrog",
                    url: "http://54.241.139.106:8082/artifactory",
                    credentialsId: "jfrog"
                )

                rtMavenDeployer (
                    id: "MAVEN_DEPLOYER",
                    serverId: "jfrog",
                    releaseRepo: "libs-release-local",
                    snapshotRepo: "libs-snapshot-local"
                )

                rtMavenResolver (
                    id: "MAVEN_RESOLVER",
                    serverId: "jfrog",
                    releaseRepo: "libs-release-local",
                    snapshotRepo: "libs-snapshot-local"
                )
            }
    }


    stage ('Publish build info') {
            steps {
                rtPublishBuildInfo (
                    serverId: "jfrog"
             )
        }
    }

    stage('Copy Dockerfile & Playbook to Ansible Server') {
            
            steps {
                  sshagent(['sshkey']) {
                       
                        sh "scp -o StrictHostKeyChecking=no Dockerfile admin@54.176.253.154:/home/admin"
                        sh "scp -o StrictHostKeyChecking=no create-container-image.yaml admin@54.176.253.154:/home/admin"
                    }
                }
            
        } 
    stage('Build Container Image') {
            
            steps {
                  sshagent(['sshkey']) {
                       
                        sh "ssh -o StrictHostKeyChecking=no admin@54.176.253.154 -C \"sudo ansible-playbook create-container-image.yaml\""
                        
                    }
                }
            
        } 
    stage('Copy Deployent & Service Defination to K8s Master') {
            
            steps {
                  sshagent(['sshkey']) {
                       
                        sh "sudo scp -o StrictHostKeyChecking=no create-k8s-deployment.yaml admin@13.56.178.241:/home/admin"
                        sh "sudo scp -o StrictHostKeyChecking=no nodePort.yaml admin@13.56.178.241:/home/admin"
                    }
                }
            
        } 


    stage('Deploy Artifacts to Production') {
            
            steps {
                  sshagent(['sshkey']) {
                       
                        sh "ssh -o StrictHostKeyChecking=no admin@13.56.178.241 -C \"sudo kubectl apply -f create-k8s-deployment.yaml\""
                        sh "ssh -o StrictHostKeyChecking=no admin@13.56.178.241 -C \"sudo kubectl apply -f nodePort.yaml\""
                        
                    }
                }
            
        } 
         
   } 
}

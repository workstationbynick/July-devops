pipeline {
  agent any
  tools {
  
  maven 'maven'
   
  }
    stages {

      stage ('Checkout SCM'){
        steps {
          checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'git', url: 'https://github.com/workstationbynick/July-devops.git']]])
        }
      }
	  
	  stage ('Build')  {
	      steps {
            dir('webapp'){
            sh "pwd"
            sh "ls -lah"
            sh "mvn package"
          }
        }
         
      }
   
     stage ('SonarQube Analysis') {
        steps {
              withSonarQubeEnv('sonar') {
                
				dir('webapp'){
                 sh 'mvn -U clean install sonar:sonar'
                }
				
              }
            }
      }

    stage ('Artifactory configuration') {
            steps {
                rtServer (
                    id: "jfrog",
                    url: "http://52.91.73.6:8082/artifactory",
                    credentialsId: "jfrog"
                )

                rtMavenDeployer (
                    id: "MAVEN_DEPLOYER",
                    serverId: "jfrog",
                    releaseRepo: "july-devops-libs-release-local",
                    snapshotRepo: "july-devops-libs-snapshot-local"
                )

                rtMavenResolver (
                    id: "MAVEN_RESOLVER",
                    serverId: "jfrog",
                    releaseRepo: "july-devops-libs-release",
                    snapshotRepo: "july-devops-libs-snapshot"
                )
            }
    }

    stage ('Deploy Artifacts') {
            steps {
                rtMavenRun (
                    tool: "MAVEN", // Tool name from Jenkins configuration
                    pom: 'webapp/pom.xml',
                    goals: 'clean install',
                    deployerId: "MAVEN_DEPLOYER",
                    resolverId: "MAVEN_RESOLVER"
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
                  sshagent(['ssh_agent']) {
                       sh "chmod 400  Nick-KP.pem"
                       sh "ls -lah"
                        sh "scp -i Nick-KP.pem -o StrictHostKeyChecking=no Dockerfile ubuntu@34.194.41.127:/home/ubuntu"
                        sh "scp -i Nick-KP.pem -o StrictHostKeyChecking=no dockerfile ubuntu@34.194.41.127:/home/ubuntu"
                        sh "scp -i Nick-KP.pem -o StrictHostKeyChecking=no deploy-dockerhub.yml ubuntu@34.194.41.127:/home/ubuntu"
                    }
                }
            
        } 

    stage('Build Container Image') {
            
            steps {
                  sshagent(['ssh_key']) {
                        sh "ssh -i Nick-KP.pem -o StrictHostKeyChecking=no ubuntu@34.194.41.127 -C \"ansible-playbook  -vvv -e build_number=${BUILD_NUMBER} deploy-dockerhub.yml\""
                        
                    }
                }
            
        } 

    stage('Copy Deployment & Service Defination to K8s Master') {
            
            steps {
                  sshagent(['ssh_key']) {
                        sh "scp -i Nick-KP.pem -o StrictHostKeyChecking=no deployment.yaml ubuntu@18.205.98.141:/home/ubuntu"
                        sh "scp -i Nick-KP.pem -o StrictHostKeyChecking=no service.yaml ubuntu@18.205.98.141:/home/ubuntu"
                    }
                }
            
        } 

    stage('Waiting for Approvals') {
            
        steps{

				input('Test Completed ? Please provide  Approvals for Prod Release ?')
			  }
            
    }     
    stage('Deploy Artifacts to Production') {
            
            steps {
                  sshagent(['ssh_key']) {
                        sh "ssh -i Itern-KP.pem -o StrictHostKeyChecking=no ubuntu@18.205.98.141 -C \"kubectl set image deployment/class-deploy2 customcontainer=adegokeobafemi/lab:${BUILD_NUMBER}\""
                        //sh "ssh -i Nick-KP.pem -o StrictHostKeyChecking=no ubuntu@18.205.98.141 -C \"kubectl delete pod class-deploy2\""
                        //sh "ssh -i Nick-KP.pem -o StrictHostKeyChecking=no ubuntu@18.205.98.141 -C \"kubectl apply -f deploy.yml\""
                        //sh "ssh -i Nick-KP.pem -o StrictHostKeyChecking=no ubuntu@18.205.98.141 -C \"kubectl apply -f service.yml\""
                        
                    }
                }
            
        } 
         
   } 
}



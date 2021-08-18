def NexusRepo
def ArtifactId
def Version

pipeline {

    agent any
    
    tools {
        maven "maven-3.8.1"
        }
        
    stages {

//Build app using MAVEN
        
        stage("Build") {
            steps {
                script {
                    sh "mvn clean package "

                }
            }
        }
        
//Push jar file to ansible server, build docker image, push image to Nexus
        
        stage('Create Artifact') {
            steps([$class: 'BapSshPromotionPublisherPlugin']) {
                
          withCredentials([
              usernamePassword(
                  credentialsId: 'nexus-creds',
                  usernameVariable: 'DOCKER_USER',
                  passwordVariable: 'DOCKER_PASSWORD'
              )
          ])
                script { 
                     NexusRepo = readMavenPom().getVersion().contains("snapshot") ? "ip-10-0-1-140.eu-central-1.compute.internal:8083/" : "ip-10-0-1-140.eu-central-1.compute.internal:8084/"
                     ArtifactId = readMavenPom().getArtifactId()
                     Version = readMavenPom().getVersion()
                sshPublisher(
                continueOnError: false, failOnError: true,
                publishers: [
                    sshPublisherDesc(
                        configName: "ansibleserver",
                        verbose: true,
                        transfers:[
                            sshTransfer(sourceFiles: "target/*.jar", removePrefix: "target", remoteDirectory: "docker-java-pet-clinic",
                            execCommand:"ansible-playbook  build-image.yml --extra-vars 'ImageVersion=${ArtifactId}-${Version}-build-${env.BUILD_NUMBER} repos=${NexusRepo}${ArtifactId}-${Version}-build-${env.BUILD_NUMBER} ImageVersionJinja=${ArtifactId}-${Version}. DOCKER_USER=${env.DOCKER_USER} DOCKER_PASSWORD=${env.DOCKER_PASSWORD}'")
                           
                                    ]
                                    )
                             ]
                         )
                                                            }
                          }
        }
   // Run playbook that deploys on Amazon ECS
        
        stage('Deploy Artifact') {
            steps([$class: 'BapSshPromotionPublisherPlugin']) {
                script {
                sshPublisher(
                continueOnError: false, failOnError: true,
                publishers: [
                    sshPublisherDesc(
                        configName: "ansibleserver",
                        verbose: true,
                        transfers:[
                            sshTransfer(
                            execCommand:"ansible-playbook  deploy-image.yml --extra-vars 'repos=${NexusRepo}${ArtifactId}-${Version}-build-${env.BUILD_NUMBER}'"
                            )
                           
                                    ]
                                    )
                             ]
                         )
                                                            }
                          }
        }
        

    }
//Clean Workspace
    
       post {
          always {
            cleanWs()
        }
    }
}

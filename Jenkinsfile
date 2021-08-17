pipeline {
    agent {
        label "master"
    }
    
    tools {
        maven "maven-3.8.1"

        }
        
    environment {
    }
    
    stages {
        stage("Clone code from VCS") {
            
            steps {
                 checkout([$class: 'GitSCM', branches: [[name: '*/main']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: '', url: 'https://github.com/psavelkov91/spring-petclinic']]]) 
            }
            
            
        }
        stage("Maven Build") {
            steps {
                script {
                    sh "mvn clean package "

                }
            }
        }
        
        
        stage('Build Image&Push to Nexus') {
            steps([$class: 'BapSshPromotionPublisherPlugin']) {
                script { 
                    def NexusRepo = readMavenPom().getVersion().contains("snapshot") ? "ip-10-0-1-140.eu-central-1.compute.internal:8083/" : "ip-10-0-1-140.eu-central-1.compute.internal:8084/"
                    def ArtifactId = readMavenPom().getArtifactId()
                    def Version = readMavenPom().getVersion()
                    def Name = readMavenPom().getName()
                    def GroupId = readMavenPom().getGroupId()
                sshPublisher(
                continueOnError: false, failOnError: true,
                publishers: [
                    sshPublisherDesc(
                        configName: "ansibleserver",
                        verbose: true,
                        transfers:[
                            sshTransfer(sourceFiles: "target/*.jar", removePrefix: "target", remoteDirectory: "docker-java-pet-clinic",
                            execCommand:"ansible-playbook  build-image.yml --extra-vars 'ImageVersion=${ArtifactId}-${Version}-build-${env.BUILD_NUMBER}  repos=${NexusRepo}${ArtifactId}-${Version}-build-${env.BUILD_NUMBER} ImageVersionJinja=${ArtifactId}-${Version}.'")
                           
                                    ]
                                    )
                             ]
                         )
                                                            }
                          }
        }

    }
}

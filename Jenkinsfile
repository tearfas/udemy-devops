pipeline{
    //Directives
    agent any
    tools {
        maven 'maven'
    }
    environment {
        ArtifactId = readMavenPom().getArtifactId()
        Version = readMavenPom().getVersion()
        Name = readMavenPom().getName()
        GroupId = readMavenPom().getGroupId()
    }

    stages {
        // Specify various stage with in stages
        // stage 1. Build
        stage ('Build'){
            steps {
                sh 'mvn clean install package'
            }
        }
        // Stage2 : Testing
        stage ('Test'){
            steps {
                echo 'testing......'

            }
        }
        
        // Stage3 : Publish the artifacts to Nexus
        stage ('Publish to Nexus'){
            steps {
                script {
                     def NexusRepo = Version.endsWith("SNAPSHOT") ? "BrightDevopsLab-SNAPSHOT" : "BrightDevopsLab-RELEASE"
                     nexusArtifactUploader artifacts: 
                     [[artifactId: "${ArtifactId}", 
                     classifier: '', 
                     file: "target/${ArtifactId}-${Version}.war", 
                     type: 'war']], 
                     credentialsId: '5c98ee06-f735-4b4b-8a9e-142d0b151fdf', 
                     groupId: "${GroupId}", 
                     nexusUrl: '192.168.1.217:8081', 
                     nexusVersion: 'nexus3', 
                     protocol: 'http', 
                     repository: "${NexusRepo}", 
                     version: "${Version}"
                }
            }
        }

        //Stage 4 : Print Variables declared above
        stage ('Print environment Variables') {
            steps {
                echo "Artifact Id is '${ArtifactId}'"
                echo "Version is '${Version}'"
                echo "Name is '${Name}'"
                echo "Group ID is '${GroupId}'"
            }
        }
    
        // Stage 5 : Publish the source code to Sonarqube
        stage ('Deploy to tomcat') {
            steps {
                echo 'Deploying to Prod...... '
                sshPublisher(publishers: 
                [sshPublisherDesc(
                    configName: 'Ansiblecontrolnode', 
                    transfers: [
                        sshTransfer (
                            cleanRemote: false,
                            execCommand: 'ansible-playbook /home/sysadmin/udemy-devops/downloaddeploy.yml -i /home/sysadmin/udemy-devops/inventory',
                            execTimeout: 1200000
                        )
                    ], 
                    usePromotionTimestamp: false, useWorkspaceInPromotion: false, 
                    verbose: false)])
            }
        } 
        // Stage 6 : Publish the source code to Docker
        stage ('Deploy to docker') {
            steps {
                echo 'Deploying to Docker...... '
                sshPublisher(publishers: 
                [sshPublisherDesc(
                    configName: 'Ansiblecontrolnode', 
                    transfers: [
                        sshTransfer (
                            cleanRemote: false,
                            execCommand: 'ansible-playbook /home/sysadmin/udemy-devops/downloaddeploy_docker.yml -i /home/sysadmin/udemy-devops/inventory',
                            execTimeout: 1200000
                        )
                    ], 
                    usePromotionTimestamp: false, useWorkspaceInPromotion: false, 
                    verbose: false)])
            }
        } 
    }

}

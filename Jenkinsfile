pipeline {
    agent any
    options { 
        // Grant permission to downstream job(s) to copy artifacts
        copyArtifactPermission("/${env.JOB_NAME}")

    }    
    stages {
        stage('BUILDING DOCKER IMAGE STAGE') {
            steps {
                script {
                    echo "env.JOB_NAME: ${env.JOB_NAME}"
                    env.DOCKER_IMAGE = "alpine:3.23.2_${BUILD_NUMBER}" 
                    writeFile file: "DOCKER_IMAGE_FILE.txt", text: env.DOCKER_IMAGE
                    archiveArtifacts artifacts: "DOCKER_IMAGE_FILE.txt", onlyIfSuccessful: true
                }
            }
        }
    }  
    post {
        always {
            script {
                // List all available properties and methods 
                // echo "=== currentBuild Properties ===" 
                // currentBuild.properties.each { k, v -> 
                //                               echo "${k} = ${v}" 
                //                               }

                // Get previous successful build number
                // def previousBuild = currentBuild.rawBuild.getPreviousSuccessfulBuild()
                def previousBuild = currentBuild.previousSuccessfulBuild

                echo "lastSuccess: ${previousBuild.number}"
                
                if (previousBuild != null) {
                    /*
                      copyArtifacts plugin should be installed
                      Pipeline config --> Permission to Copy Artifact --> Projects to allow copy artifacts
                      Example: If you have a job called sourceproject that archives files with archiveArtifacts, 
                      then another job can run copyArtifacts(projectName: 'sourceproject') 
                      to pull those files into its own workspace

                      If your jobs are organized inside folders, you must include the folder path. For example:
                      projectName: 'MyFolder/sourceproject'
                    */

                    // Get the artifact from previous build 
                    def previousImageFile = copyArtifacts( projectName: "${env.JOB_NAME}",                                                        
                                                           // selector: [$class: 'SpecificBuildSelector', buildNumber: "${previousBuild.number}"],
                                                           // selector: lastSuccessful(),  
                                                           selector: specific("${previousBuild.number}"),                                                                                                                                        
                                                           filter: 'DOCKER_IMAGE_FILE.txt', 
                                                           target: 'artifacts/' )
                                                                             
                    echo "previousImageFile: ${previousImageFile}"
                    // sh 'cat artifacts/DOCKER_IMAGE_FILE.txt'
                    def previousFile = readFile(file: "artifacts/DOCKER_IMAGE_FILE.txt")
                    echo "previousFile: ${previousFile}"
                }
                else {
                    echo "No previous successful build found. Cannot rollback."
                }
            } 
        } 
    } 
} 
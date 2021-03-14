@Library('javahome-libs') _

pipeline{
    agent any
    options {
      timeout(30)
    }
    stages{
        
        stage('Maven and Sonar'){
            
            parallel{
            stage('Sonar Analysis'){
                steps{
                    withSonarQubeEnv('sonar7') {
                        sh 'mvn sonar:sonar'
                    }
                    
                    timeout(time: 1, unit: 'HOURS') {
                        script{
                          def qg = waitForQualityGate()
                          if (qg.status != 'OK') {
                              error "Pipeline aborted due to quality gate failure: ${qg.status}"
                          }
                        }
                  }
                }
            }
            
             stage('Mvn Build'){
                steps{
                    sh 'mvn clean package'
                }
            }
        
     
        }
        
        }
        stage("Nexus Deploy"){
            steps{
                script{
                    def pomFile = readMavenPom file: 'pom.xml'
                    nexusArtifactUploader artifacts: [[artifactId: 'myweb', classifier: '', file: "target/myweb-${pomFile.version}.war", type: 'war']], 
                                      credentialsId: 'nexus3', 
                                      groupId: 'in.javahome', 
                                      nexusUrl: '172.31.15.174:8081',
                                      nexusVersion: 'nexus3', 
                                      protocol: 'http', repository: 'my-app', 
                                      version: '0.0.12'
                }
            }
        }
          
        stage('Tomcat Deploy'){
            steps{
                tomcatDeploy("ec2-user","172.31.43.126","myweb")
            }
        }
     
    }
}

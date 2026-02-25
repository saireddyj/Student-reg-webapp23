pipeline {
    
    agent any
    
    triggers {
       githubPush()
    }

    options {
        buildDiscarder logRotator(numToKeepStr: '5')
        disableConcurrentBuilds()
        timeout(time: 10,unit: 'MINUTES')
    }
 
    environment {
        
        SONARQUBE_HOST = "http://172.31.26.88:9000"
        SONARQUBE_TOKEN = credentials('SonarQubeToken')
        tomcatserverSSHUserName = "ec2-user"
        tomcatSystemIP = "172.31.42.177"
        
    }
 
    tools {
        maven 'Maven-3.9.11'
    }
    
    stages{

        stage("Build Stage"){
           
         steps {
                sh "mvn clean package"
           }
        }
     
        stage("Sonar Scan"){
            steps {
                sh "${maven_home}/bin/mvn clean verify sonar:sonar -Dsonar.token=sqa_053bb385c0b1526c3612fb59efc2478daa63f7ba"
            }
        }
     
        stage("Upload Artificat To Nexus"){
            steps {
                sh "mvn clean deploy"
            }
        }
     
        stage("Deploy War File To Tomcat"){
            
            when {
                expression { env.BRANCH_NAME ==  "development" }
            }

            steps {
                sshagent(['TomcatServer_SSH_Credetails']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${tomcatserverSSHUserName}@${tomcatSystemIP} sudo systemctl stop tomcat
                        sleep 20
                        ssh -o StrictHostKeyChecking=no ${tomcatserverSSHUserName}@${tomcatSystemIP} rm /opt/tomcat/webapps/student-reg-webapp.war || true
                        scp -o StrictHostKeyChecking=no target/student-reg-webapp.war ${tomcatserverSSHUserName}@${tomcatSystemIP}:/opt/tomcat/webapps/student-reg-webapp.war
                        ssh -o StrictHostKeyChecking=no ${tomcatserverSSHUserName}@${tomcatSystemIP} sudo systemctl start tomcat
                        """
                }
            }
        }

        stage("Deploy War File To QA Server"){
            
            
            when {
                expression { env.BRANCH_NAME ==  "QA" }
            }

            steps {
                sshagent(['TomcatServer_SSH_Credetails']) {
                  sh """
                    echo "Deploying to QA Server"
                    """
                }
            }
        }
  
       stage("Deploy War File To Prod Server"){
             
            when {
                expression { env.BRANCH_NAME ==  "main" }
            }
            steps {
              sshagent(['TomcatServer_SSH_Credetails']) {
                sh """
                  echo "Deploying to Prod Server"
                    """
              }
            }
        }

    } 


    post {
        success {
            slackSend channel: 'lic-app-team', color: "good", message: "Jenkins Job ${env.JOB_NAME} - ${env.BUILD_NUMBER} - Success . Please heck console output at ${env.BUILD_URL}"  
        }
       
        failure {
            
            slackSend channel: 'lic-app-team', color: "danager", message: "Jenkins Job ${env.JOB_NAME} - ${env.BUILD_NUMBER} - Failed . Please Check console output at ${env.BUILD_URL}"  
        }
       
        always {
            cleanWs()
        }
    }

}

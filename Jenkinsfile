pipeline {
    agent any
    environment {
     Sonar_URL = "http://3.110.128.88:9000"
     SonarToken = credentials("sonartoken")
     Tomcat_IP = "35.154.205.53"
     }
    tools {
           maven 'Maven3.9.10'
    }
    stages {
        stage("Git Clone") {
            steps{
             git branch: 'devolopment', credentialsId: 'GitHubcred', url: 'https://github.com/abhiGithubIT/student-reg-webapp.git'
            }
        }
        stage('maven clean package') {
            steps {
                  sh "mvn clean package"
            }
        }
        stage("Sonar Scan") {
            steps{
                  sh "mvn  clean verify sonar:sonar -Dsonar.url={Sonar_URL} -Dsonar.token=${sonartoken}"
            }
        }
        stage("Uploading The War file to nexus") {
            steps {
                sh "mvn clean deploy"
            }
        }
        stage("Stop Tomcat Process") {
            steps{
                sshagent(['tomcatcredential']) {
                sh """
                  echo stoping the Tomcat Process
                  ssh -o StrictHostKeyChecking=no ec2-user@${Tomcat_IP} sudo systemctl stop tomcat
                  sleep 10
                  """
                 }
            }
        }
         stage("deplyoing war file to tomcat") {
              steps{
                   sshagent(['tomcatcredential']) {
                    sh "scp -o StrictHostKeyChecking=no target/student-reg-webapp.war ec2-user@${Tomcat_IP}:/opt/tomcat/webapps/"
                     }
              }
         }
          stage("Starting The Tomcat Process") {
              steps{
                 sshagent(['tomcatcredential']) {
                  sh """
                    echo starting the tomcat Process
                    ssh -o StrictHostKeyChecking=no ec2-user@${Tomcat_IP} sudo systemctl start tomcat
                    """
                    }
              }
         }

pipeline {
    agent any
    environment {
     Sonar_URL = "http://3.110.133.51:9000"
     SonarToken = credentials("sonartoken")
     Tomcat_IP = "13.233.9.125"
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
        stage("Deploy to dev server") {
            when {
                expression { 
                    branch = 'devolopment'
                 }
            }
             steps{
                sshagent(['tomcatcredential']) {
                sh """
                  echo stoping the Tomcat Process
                  ssh -o StrictHostKeyChecking=no ec2-user@${Tomcat_IP} sudo systemctl stop tomcat
                  sleep 30
                  scp -o StrictHostKeyChecking=no target/student-reg-webapp.war ec2-user@${Tomcat_IP}:/opt/tomcat/webapps/student-reg-webapp.war
                  ssh -o StrictHostKeyChecking=no ec2-user@${Tomcat_IP} sudo systemctl start tomcat
                  echo starting the tomcat Process
                  """
                 }
            }
        }
        stage("Deploy to feature-login server") {
            when {
                expression { 
                    branch = 'feature-login'
                 }
            }
             steps{
                sshagent(['tomcatcredential']) {
                sh """
                  echo stoping the Tomcat Process
                  ssh -o StrictHostKeyChecking=no ec2-user@${Tomcat_IP} sudo systemctl stop tomcat
                  sleep 30
                  scp -o StrictHostKeyChecking=no target/student-reg-webapp.war ec2-user@${Tomcat_IP}:/opt/tomcat/webapps/student-reg-webapp.war
                  ssh -o StrictHostKeyChecking=no ec2-user@${Tomcat_IP} sudo systemctl start tomcat
                  echo starting the tomcat Process
                  """
                 }
            }
        }
        
    } 
}


post {
    success {
        script {
            emailext(
                to: 'abhiabhishek299@gmail.com',
                subject: "${env.JOB_NAME} Build #${env.BUILD_NUMBER} - SUCCESS",
                body: "Build was successful. Please check the console output at ${env.BUILD_URL}"
            )
        }
    }
    failure {
        script {
            emailext(
                to: 'abhiabhishek299@gmail.com',
                subject: "${env.JOB_NAME} Build #${env.BUILD_NUMBER} - FAILURE",
                body: "Build failed. Please check the console output at ${env.BUILD_URL}"
            )
        }
    }
}


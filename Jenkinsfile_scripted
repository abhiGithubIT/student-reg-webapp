node {
    def mavenHome = tool name: 'Maven3.9.10', type: 'maven'
    try { 
    stage("git clone") {
        git branch: 'devolopment', credentialsId: 'GitHubcred', url: 'https://github.com/abhiGithubIT/student-reg-webapp.git'
    }
    stage("maven verify and sonar scan") {
        withCredentials([string(credentialsId: 'sonartoken', variable: 'sonartoken')]) {
        sh "${mavenHome}/bin/mvn clean verify sonar:sonar -Dsonar.token=${sonartoken}" 
    }
    }
    stage("maven deploy") {
        sh "${mavenHome}/bin/mvn  clean  deploy" 
    }
    
    stage("stop tomcat server") {
        sshagent(['tomcatcredential']) {
            sh """
                  echo stoping the tomcat service
                  ssh -o StrictHostKeyChecking=no ec2-user@13.204.67.112 sudo systemctl stop tomcat
                  sleep 10
                  """
        }
    }
    stage("Deploy war file to tomcat") {
        sshagent(['tomcatcredential']) {
           sh  "scp -o StrictHostKeyChecking=no target/student-reg-webapp.war ec2-user@13.204.67.112:/opt/tomcat/webapps/"
    }
    }
    
    stage("starting tomcat service") {
        sshagent(['tomcatcredential']) {
            sh """
                  echo starting the tomcat service
                  ssh -o StrictHostKeyChecking=no ec2-user@13.204.67.112 sudo systemctl start tomcat
                  """
    }
    }
} 
    catch (err) {
        echo "An error occurred: ${e.getMessage()}"
        currentBuild.result = 'FAILURE'
    }
    finally {
        def buildStatus = currentBuild.result ?: 'SUCCESS'
        sendEmail(
             "${env.JOB_NAME} - Build #${env.BUILD_NUMBER} - ${buildStatus}",
             "Build ${buildStatus}. Please check the console output at ${env.BUILD_URL}",
             "abhiabhishek299@gmail.com"
        )

    }

}


def sendEmail(String subject, String body, String recipient) {
    emailext(
        subject: subject,
        body: body,
        to: recipient,
        mimeType: 'text/html'
    )
}

node {
    def mavenHome = tool name: 'Maven3.9.10', type: 'maven'
    stage("git clone") {
        git branch: 'devolopment', credentialsId: 'GitHubcred', url: 'https://github.com/abhiGithubIT/student-reg-webapp.git'
    }
    stage("maven build") {
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
                  ssh -o StrictHostKeyChecking=no ec2-user@13.203.97.104 sudo systemctl stop tomcat
                  sleep 10
                  """
        }
    }
    stage("Deploy war file to tomcat") {
        sshagent(['tomcatcredential']) {
           sh  "scp -o StrictHostKeyChecking=no target/student-reg-webapp.war ec2-user@13.203.97.104:/opt/tomcat/webapps/"
    }
    }
    
    stage("starting tomcat service") {
        sshagent(['tomcatcredential']) {
            sh """
                  echo starting the tomcat service
                  ssh -o StrictHostKeyChecking=no ec2-user@13.203.97.104 sudo systemctl start tomcat
                  """
    }
    }
}

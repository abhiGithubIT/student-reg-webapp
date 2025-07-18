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
            steps {
                git branch: 'devolopment', credentialsId: 'GitHubcred', url: 'https://github.com/abhiGithubIT/student-reg-webapp.git'
            }
        }

        stage('Maven Clean Package') {
            steps {
                sh "mvn clean package"
            }
        }

        stage("Sonar Scan") {
            steps {
                sh "mvn clean verify sonar:sonar -Dsonar.host.url=${Sonar_URL} -Dsonar.token=${SonarToken}"
            }
        }

        stage("Upload WAR to Nexus") {
            steps {
                sh "mvn clean deploy"
            }
        }

        stage("Deploy to Dev Server") {
            when {
                expression {
                    return env.BRANCH_NAME == 'devolopment'
                }
            }
            steps {
                sshagent(['tomcatcredential']) {
                    sh """
                        echo Stopping Tomcat on Dev Server...
                        ssh -o StrictHostKeyChecking=no ec2-user@${Tomcat_IP} sudo systemctl stop tomcat
                        sleep 30
                        echo Copying WAR file...
                        scp -o StrictHostKeyChecking=no target/student-reg-webapp.war ec2-user@${Tomcat_IP}:/opt/tomcat/webapps/student-reg-webapp.war
                        echo Starting Tomcat...
                        ssh -o StrictHostKeyChecking=no ec2-user@${Tomcat_IP} sudo systemctl start tomcat
                    """
                }
            }
        }

        stage("Deploy to Feature-Login Server") {
            when {
                expression {
                    return env.BRANCH_NAME == 'feature-login'
                }
            }
            steps {
                sshagent(['tomcatcredential']) {
                    sh """
                        echo Stopping Tomcat on Feature-Login Server...
                        ssh -o StrictHostKeyChecking=no ec2-user@${Tomcat_IP} sudo systemctl stop tomcat
                        sleep 30
                        echo Copying WAR file...
                        scp -o StrictHostKeyChecking=no target/student-reg-webapp.war ec2-user@${Tomcat_IP}:/opt/tomcat/webapps/student-reg-webapp.war
                        echo Starting Tomcat...
                        ssh -o StrictHostKeyChecking=no ec2-user@${Tomcat_IP} sudo systemctl start tomcat
                    """
                }
            }
        }
    }

    post {
        success {
            emailext(
                subject: "${env.JOB_NAME} - Build #${env.BUILD_NUMBER} - SUCCESS",
                body: "✅ Build was successful. View console output at: ${env.BUILD_URL}",
                to: 'abhiabhishek299@gmail.com'
            )
        }

        failure {
            emailext(
                subject: "${env.JOB_NAME} - Build #${env.BUILD_NUMBER} - FAILURE",
                body: "❌ Build failed. Please check the console output: ${env.BUILD_URL}",
                to: 'abhiabhishek299@gmail.com'
            )
        }
    }
}

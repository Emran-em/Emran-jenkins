pipeline {
    agent any

    tools {
        maven 'MAVEN_HOME' // Must match your Global Tool Configuration
    }

    environment {
        SONARQUBE = credentials('sonarqube-token')
        NEXUS_CREDENTIALS = credentials('Nexus_server')
        SLACK_TOKEN = credentials('slack')
        NEXUS_URL = 'http://52.23.219.98:8081/repository/devops/'
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/mubeen-hub78/mub_simplecutomerapp.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('MySonar') {
                    sh 'mvn clean verify sonar:sonar -Dsonar.token=$SONARQUBE'
                }
            }
        }

        stage('Build WAR') {
            steps {
                sh 'mvn package'
            }
        }

        stage('Upload to Nexus') {
            steps {
                sh """
                    mvn deploy:deploy-file \\
                      -DgroupId=com.customer \\
                      -DartifactId=simple-customer-app \\
                      -Dversion=1.0 \\
                      -Dpackaging=war \\
                      -Dfile=target/*.war \\
                      -DrepositoryId=nexus \\
                      -Durl=$NEXUS_URL \\
                      -Dusername=${NEXUS_CREDENTIALS_USR} \\
                      -Dpassword=${NEXUS_CREDENTIALS_PSW}
                """
            }
        }

        stage('Notify Slack') {
            steps {
                slackSend channel: '#devops-alerts', tokenCredentialId: 'slack', message: "✅ *Build SUCCESS:* ${env.JOB_NAME} #${env.BUILD_NUMBER}"
            }
        }
    }

    post {
        failure {
            slackSend channel: '#devops-alerts', tokenCredentialId: 'slack', message: "❌ *Build FAILED:* ${env.JOB_NAME} #${env.BUILD_NUMBER}"
        }
    }
}

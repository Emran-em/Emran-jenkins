pipeline {
    agent any

    tools {
        maven 'MAVEN_HOME' // Ensure this Maven tool is configured in Jenkins globally
    }

    environment {
        SONARQUBE = 'MySonar'                           // SonarQube server name
        NEXUS_CREDENTIALS = credentials('Nexus_server')// Nexus username/password credential ID
        NEXUS_URL = 'http://3.92.29.53:8081/repository/devops/' // Correct Nexus repo URL
        SLACK_TOKEN = credentials('slack')             // Slack token credential ID
        SLACK_CHANNEL = '#new-channel'                  // Slack channel for notifications
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/mubeen-hub78/mub_simplecutomerapp.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONARQUBE}") {
                    sh 'mvn clean verify sonar:sonar'
                }
            }
        }

        stage('Build WAR') {
            steps {
                sh 'mvn package -DskipTests'
            }
        }

        stage('Upload to Nexus') {
            steps {
                script {
                    def warFile = sh(script: "ls target/*.war", returnStdout: true).trim()
                    sh """
                        mvn deploy:deploy-file \
                        -DgroupId=com.hiring \
                        -DartifactId=hiring-app \
                        -Dversion=1.0.0 \
                        -Dpackaging=war \
                        -Dfile=${warFile} \
                        -DrepositoryId=nexus \
                        -Durl=${env.NEXUS_URL} \
                        -DgeneratePom=true \
                        -DrepositoryLayout=default \
                        -Dusername=${env.NEXUS_CREDENTIALS_USR} \
                        -Dpassword=${env.NEXUS_CREDENTIALS_PSW}
                    """
                }
            }
        }
    }

    post {
        always {
            script {
                def buildStatus = currentBuild.currentResult ?: 'SUCCESS'
                def message = "${buildStatus}: Job ${env.JOB_NAME} #${env.BUILD_NUMBER} (<${env.BUILD_URL}|Details>)"
                slackSend(channel: env.SLACK_CHANNEL, tokenCredentialId: 'slack', message: message)
            }
        }
    }
}

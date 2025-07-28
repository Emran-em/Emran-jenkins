pipeline {
    agent any

    tools {
        maven 'MAVEN_HOME'  // Make sure Maven tool is configured with this name in Jenkins
    }

    environment {
        SONARQUBE = 'MySonar'                            // Your SonarQube server name configured in Jenkins
        NEXUS_CREDENTIALS = credentials('Nexus_server') // Nexus username/password credential ID
        NEXUS_URL = 'http://52.23.219.98:8081/repository/devops/'
        SLACK_TOKEN = credentials('slack')              // Slack token credential ID
        SLACK_CHANNEL = '#new-channel'                   // Updated Slack channel here
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
                sh '''
                    mvn deploy:deploy-file \
                    -DgroupId=com.hiring \
                    -DartifactId=hiring-app \
                    -Dversion=1.0.0 \
                    -Dpackaging=war \
                    -Dfile=target/*.war \
                    -DrepositoryId=nexus \
                    -Durl=$NEXUS_URL \
                    -DgeneratePom=true \
                    -DrepositoryLayout=default \
                    -Dusername=$NEXUS_CREDENTIALS_USR \
                    -Dpassword=$NEXUS_CREDENTIALS_PSW
                '''
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

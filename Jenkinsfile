pipeline {
    agent any

    tools {
        maven 'MAVEN_HOME'
    }

    environment {
        SONARQUBE_URL = 'http://52.23.219.98:9000'
        SONAR_TOKEN = credentials('sonarqube-token')
        NEXUS_URL = 'http://52.23.219.98:8081/repository/devops/'
        NEXUS_CRED = credentials('Nexus_server')
        SLACK_TOKEN = credentials('slack')
    }

    stages {
        stage('Checkout Code') {
            steps {
                git url: 'https://github.com/mubeen-hub78/mub_simplecutomerapp.git'
            }
        }

        stage('Build & SonarQube Analysis') {
            steps {
                sh '''
                    mvn clean install sonar:sonar \
                    -Dsonar.host.url=$SONARQUBE_URL \
                    -Dsonar.login=$SONAR_TOKEN
                '''
            }
        }

        stage('Upload Artifact to Nexus') {
            steps {
                sh '''
                    mvn deploy -DaltDeploymentRepository=devops-repo::default::${NEXUS_URL} \
                    -Dnexus.username=${NEXUS_CRED_USR} \
                    -Dnexus.password=${NEXUS_CRED_PSW}
                '''
            }
        }
    }

    post {
        success {
            slackSend(channel: '#general', color: 'good', message: "✅ Job '${env.JOB_NAME}' build #${env.BUILD_NUMBER} succeeded!")
        }
        failure {
            slackSend(channel: '#general', color: 'danger', message: "❌ Job '${env.JOB_NAME}' build #${env.BUILD_NUMBER} failed!")
        }
    }
}

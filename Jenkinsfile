pipeline {
    agent any

    tools {
        maven 'Maven3'
    }

    environment {
        SONARQUBE = 'MySonar'
        NEXUS_CREDS = credentials('Nexus_server')
        SONAR_TOKEN = credentials('sonarqube-token')
        SLACK_TOKEN = credentials('slack')
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/mubeen-hub78/mub_simplecutomerapp.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONARQUBE}") {
                    sh """
                        mvn clean verify sonar:sonar \
                        -Dsonar.projectKey=SimpleCustomerAppKey:SimpleCustomerApp \
                        -Dsonar.projectName=SimpleCustomerApp \
                        -Dsonar.projectVersion=2.0 \
                        -Dsonar.login=${SONAR_TOKEN}
                    """
                }
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn package'
            }
        }

        stage('Upload to Nexus') {
            steps {
                sh """
                    mvn deploy -DaltDeploymentRepository=internal.repo::default::http://52.23.219.98:8081/repository/maven-releases/ \
                    -Dnexus.username=${NEXUS_CREDS_USR} -Dnexus.password=${NEXUS_CREDS_PSW}
                """
            }
        }
    }

    post {
        success {
            slackSend (tokenCredentialId: 'slack', channel: '#devops', message: "✅ Pipeline succeeded for *SimpleCustomerApp*", color: 'good')
        }
        failure {
            slackSend (tokenCredentialId: 'slack', channel: '#devops', message: "❌ Pipeline failed for *SimpleCustomerApp*", color: 'danger')
        }
    }
}

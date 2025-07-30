pipeline {
    agent any

    tools {
        maven 'MAVEN_HOME'
    }

    environment {
        SONARQUBE = 'MySonar'
        NEXUS_URL = 'http://34.204.71.153:8081/repository/devops/'
        SLACK_CHANNEL = '#new-channel'
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
                    // IMPORTANT: Hardcoding credentials directly like this is INSECURE for production.
                    // This is for debugging purposes only to isolate the authentication issue.
                    sh """
                        mvn deploy:deploy-file \\
                        -DgroupId=com.javatpoint \\
                        -DartifactId=SimpleCustomerApp \\
                        -Dversion=1.0.0-SNAPSHOT \\
                        -Dpackaging=war \\
                        -Dfile=${warFile} \\
                        -DrepositoryId=nexus \\
                        -Durl=${NEXUS_URL} \\
                        -DgeneratePom=true \\
                        -Dusername=admin \\
                        -Dpassword=123456
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

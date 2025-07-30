pipeline {
    agent any

    tools {
        maven 'MAVEN_HOME'
    }

    environment {
        SONARQUBE = 'MySonar'
        NEXUS_CREDENTIALS = credentials('Nexus_server')
        NEXUS_URL = 'http://3.92.29.53:8081/repository/maven-releases/'
        SLACK_TOKEN = credentials('slack')
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
                    // Get the correct WAR file name based on your artifactId and version:
                    def warFile = sh(script: "ls target/SimpleCustomerApp*.war", returnStdout: true).trim()
                    sh """
                        mvn deploy:deploy-file \\
                        -DgroupId=com.javatpoint \\
                        -DartifactId=SimpleCustomerApp \\
                        -Dversion=1.0.0-SNAPSHOT \\
                        -Dpackaging=war \\
                        -Dfile=${warFile} \\
                        -DrepositoryId=nexus \\
                        -Durl=${env.NEXUS_URL} \\
                        -DgeneratePom=true \\
                        -DrepositoryLayout=default \\
                        -Dusername=${env.NEXUS_CREDENTIALS_USR} \\
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

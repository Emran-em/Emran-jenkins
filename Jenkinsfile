pipeline {
    agent any

    tools {
        maven 'MAVEN_HOME'
    }

    environment {
        SONARQUBE = 'MySonar'
        NEXUS_CREDENTIALS = credentials('Nexus_server') // ✅ Using correct ID
        NEXUS_URL = 'http://3.85.130.86:8081/repository/devops/' // ✅ Updated from maven-releases to devops
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
                        -Dusername=${NEXUS_CREDENTIALS_USR} \\
                        -Dpassword=${NEXUS_CREDENTIALS_PSW}
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

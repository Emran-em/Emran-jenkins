pipeline {
    agent {
        docker {
            image 'maven:3.8.7-eclipse-temurin-17'
        }
    }

    tools {
        maven 'MAVEN_HOME'
    }

    environment {
        SONARQUBE = 'MySonar'
        NEXUS_CREDENTIALS = credentials('nexus')
        NEXUS_URL = 'http://52.23.219.98:8081/repository/devops/'
    }

    stages {
        stage('Git Clone') {
            steps {
                git branch: 'main', url: 'https://github.com/betawins/hiring-app.git'
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

        stage('Slack Notification') {
            steps {
                slackSend (channel: '#devops', message: "Build Completed: ${env.JOB_NAME} #${env.BUILD_NUMBER}")
            }
        }
    }
}

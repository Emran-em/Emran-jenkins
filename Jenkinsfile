pipeline {
    agent any

    environment {
        SONAR_HOST = 'http://13.217.24.93:9000'
        SONAR_TOKEN_CREDENTIAL_ID = 'sonar'   // your Sonar token credential id
        NEXUS_URL = 'http://13.217.24.93:8081/repository/maven-snapshots/in/'
        NEXUS_USERNAME = 'admin'
        NEXUS_PASSWORD = 'Mubsad321.'
        SLACK_WEBHOOK_URL = 'https://hooks.slack.com/services/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'
        TOMCAT_URL = 'http://13.217.24.93:8082/manager/text'
        TOMCAT_USERNAME = 'admin'
        TOMCAT_PASSWORD = 'admin123'
        APP_CONTEXT = 'simplecustomerapp'
        GIT_REPO = 'https://github.com/mubeen-hub78/mub_simplecutomerapp.git'
        GIT_BRANCH = 'master'
    }

    stages {
        stage('Git Clone') {
            steps {
                echo 'Cloning repository...'
                git branch: "${env.GIT_BRANCH}", url: "${env.GIT_REPO}"
            }
        }

        stage('Build with Maven') {
            steps {
                echo 'Building with Maven inside Docker...'
                sh '''
                  docker run --rm \
                    -v "$PWD":/usr/src/mymaven \
                    -w /usr/src/mymaven \
                    maven:3.8.6-eclipse-temurin-17 \
                    mvn clean package -DskipTests
                '''
            }
        }

        stage('SonarQube Analysis') {
            steps {
                echo 'Running SonarQube analysis...'
                withCredentials([string(credentialsId: "${env.SONAR_TOKEN_CREDENTIAL_ID}", variable: 'SONAR_TOKEN')]) {
                    sh '''
                      BIN_DIR=$(find target -type d -path "*/WEB-INF/classes" | head -n 1)

                      if [ -z "$BIN_DIR" ]; then
                        echo "No compiled classes found for Sonar analysis."
                        exit 1
                      fi

                      docker run --rm \
                        -e SONAR_HOST_URL=${SONAR_HOST} \
                        -e SONAR_TOKEN=${SONAR_TOKEN} \
                        -v "$PWD":/usr/src \
                        sonarsource/sonar-scanner-cli \
                        -Dsonar.projectKey=simplecustomerapp \
                        -Dsonar.sources=src \
                        -Dsonar.java.binaries="$BIN_DIR" \
                        -Dsonar.host.url=${SONAR_HOST} \
                        -Dsonar.login=${SONAR_TOKEN}
                    '''
                }
            }
        }

        stage('Upload to Nexus') {
            steps {
                echo 'Uploading artifact to Nexus...'
                sh '''
                  WAR_FILE=$(find target -name "*.war" | head -n 1)

                  if [ ! -f "$WAR_FILE" ]; then
                    echo "WAR file not found, build may have failed."
                    exit 1
                  fi

                  curl -v -u ${NEXUS_USERNAME}:${NEXUS_PASSWORD} --upload-file "$WAR_FILE" \
                    ${NEXUS_URL}${APP_CONTEXT}/0.1-SNAPSHOT/${APP_CONTEXT}-0.1-SNAPSHOT.war
                '''
            }
        }

        stage('Slack Notification') {
            steps {
                echo 'Sending Slack notification...'
                sh """
                  curl -X POST -H 'Content-type: application/json' \
                    --data '{"text":"✅ Build SUCCESSFUL for *${APP_CONTEXT}*! 🚀"}' \
                    ${SLACK_WEBHOOK_URL}
                """
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                echo 'Deploying WAR to Tomcat...'
                sh '''
                  WAR_FILE=$(find target -name "*.war" | head -n 1)

                  curl -T "$WAR_FILE" \
                    "${TOMCAT_URL}/deploy?path=/${APP_CONTEXT}&update=true" \
                    --user ${TOMCAT_USERNAME}:${TOMCAT_PASSWORD}
                '''
            }
        }
    }
}

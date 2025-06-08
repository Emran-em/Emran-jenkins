pipeline {
    agent any

    environment {
        SONAR_HOST = 'http://52.23.219.98:9000'
        SONAR_TOKEN_CREDENTIAL_ID = 'sonar'
        NEXUS_URL = 'http://52.23.219.98:8081/repository/maven-snapshots/'
        NEXUS_USERNAME = 'admin'
        NEXUS_PASSWORD = 'Mubsad321.'
        SLACK_WEBHOOK_URL = 'https://hooks.slack.com/services/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'
        TOMCAT_URL = 'http://52.23.219.98:8083/manager/text'
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
                script {
                    def mavenContainerName = "maven-build-${UUID.randomUUID()}"
                    def containerProjectRoot = "/app"

                    sh """
                        docker run -d --name ${mavenContainerName} \
                          -v "$PWD":"${containerProjectRoot}" \
                          -v /var/lib/jenkins/.m2:/root/.m2 \
                          -w "${containerProjectRoot}" \
                          maven:3.8.6-eclipse-temurin-17 \
                          mvn clean compile package -DskipTests
                    """

                    sh "docker logs -f ${mavenContainerName} & PID=\$!; docker wait ${mavenContainerName}; kill \$PID || true"

                    sh """
                        # Check if target exists in container before copying (to avoid docker cp errors if build failed)
                        docker exec ${mavenContainerName} ls "${containerProjectRoot}/target" > /dev/null 2>&1
                        if [ \$? -ne 0 ]; then
                            echo "WARNING: target directory not found in container. Maven build likely failed."
                            exit 1
                        fi
                        docker cp ${mavenContainerName}:${containerProjectRoot}/target ./target || { echo "ERROR: Failed to copy target directory from container!"; exit 1; }
                    """

                    sh "docker rm ${mavenContainerName} || true"
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                echo 'Running SonarQube analysis...'
                withCredentials([string(credentialsId: "${env.SONAR_TOKEN_CREDENTIAL_ID}", variable: 'SONAR_TOKEN')]) {
                    def containerProjectRoot = "/app"

                    sh '''
                      docker run --rm \
                        -e SONAR_HOST_URL=${SONAR_HOST} \
                        -e SONAR_TOKEN=${SONAR_TOKEN} \
                        -v "$PWD":"${containerProjectRoot}" \
                        -w "${containerProjectRoot}" \
                        sonarsource/sonar-scanner-cli \
                        -Dsonar.projectKey=${APP_CONTEXT} \
                        -Dsonar.sources=src \
                        -Dsonar.java.binaries=target/classes \
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
                      echo "ERROR: WAR file not found, build may have failed."
                      exit 1
                    fi

                    curl -v -u ${NEXUS_USERNAME}:${NEXUS_PASSWORD} --upload-file "$WAR_FILE" \
                      ${NEXUS_URL}${APP_CONTEXT}/0.1-SNAPSHOT/${APP_CONTEXT}-0.1-SNAPSHOT.war || { echo "ERROR: Nexus upload failed!"; exit 1; }
                '''
            }
        }

        stage('Slack Notification') {
            steps {
                echo 'Sending Slack notification...'
                sh """
                    curl -X POST -H 'Content-type: application/json' \
                      --data '{"text":"✅ *Build SUCCESSFUL* for *${APP_CONTEXT}* on *${GIT_BRANCH}* branch! 🚀"}' \
                      ${SLACK_WEBHOOK_URL} || { echo "WARNING: Slack notification failed!"; }
                """
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                echo 'Deploying WAR to Tomcat...'
                sh '''
                    WAR_FILE=$(find target -name "*.war" | head -n 1)

                    if [ ! -f "$WAR_FILE" ]; then
                      echo "ERROR: WAR file not found for deployment."
                      exit 1
                    fi

                    curl -T "$WAR_FILE" \
                      "${TOMCAT_URL}/deploy?path=/${APP_CONTEXT}&update=true" \
                      --user ${TOMCAT_USERNAME}:${TOMCAT_PASSWORD} || { echo "ERROR: Tomcat deployment failed!"; exit 1; }
                '''
            }
        }
    }
}

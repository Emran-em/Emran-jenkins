pipeline {
    agent any

    environment {
        // Updated all IPs to 52.23.219.98 for consistency
        SONAR_HOST = 'http://52.23.219.98:9000'
        SONAR_TOKEN_CREDENTIAL_ID = 'sonar'
        // Corrected NEXUS_URL to a more standard path
        NEXUS_URL = 'http://52.23.219.98:8081/repository/maven-snapshots/'
        NEXUS_USERNAME = 'admin'
        NEXUS_PASSWORD = 'Mubsad321.'
        // Updated Slack Webhook URL
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
                // DEBUG: Verify initial workspace content on Jenkins host
                sh 'echo "--- Initial Workspace Content (Jenkins Host) ---"'
                sh 'ls -l'
                sh 'echo "--- Contents of src directory (Jenkins Host) ---"'
                sh 'ls -l src'
                sh 'echo "--- Finding all .java files in src directory (Jenkins Host) ---"'
                sh 'find src -name "*.java" || echo "No .java files found in src." '
            }
        }

        stage('Build with Maven') {
            steps {
                echo 'Building with Maven inside Docker...'
                script { // Using script block for advanced shell logic and variable handling
                    def mavenContainerName = "maven-build-${UUID.randomUUID()}"
                    // Define a consistent internal path for your project inside Docker containers
                    def containerProjectRoot = "/app"

                    // DEBUG: Verify contents at the root of the mounted project inside the Maven container *before* the main build command
                    sh """
                        echo "--- Contents at root of mounted project (${containerProjectRoot}) inside Maven container (pre-build check) ---"
                        docker run --rm \
                          -v "$PWD":"${containerProjectRoot}" \
                          -w "${containerProjectRoot}" \
                          maven:3.8.6-eclipse-temurin-17 \
                          ls -l "${containerProjectRoot}" || { echo "ERROR: Mounted path not found or empty inside Maven container!"; exit 1; }

                        echo "--- Running Maven build ---"
                        docker run -d --name ${mavenContainerName} \
                          -v "$PWD":"${containerProjectRoot}" \
                          -v /var/lib/jenkins/.m2:/root/.m2 \
                          -w "${containerProjectRoot}" \
                          maven:3.8.6-eclipse-temurin-17 \
                          mvn clean compile package -DskipTests
                    """

                    // Wait for the Maven command inside the container to finish and stream its logs
                    // The 'kill $PID' might report 'No such process' if the log stream finished before kill
                    // This is generally harmless and can be ignored for now.
                    sh "docker logs -f ${mavenContainerName} & PID=\$!; docker wait ${mavenContainerName}; kill \$PID || true"

                    // Copy the 'target' directory from container to Jenkins host workspace
                    sh """
                        echo "--- Copying target directory from Maven container to Jenkins workspace ---"
                        # Check if target exists in container before copying (to avoid docker cp errors if build failed)
                        docker exec ${mavenContainerName} ls "${containerProjectRoot}/target" > /dev/null 2>&1
                        if [ \$? -ne 0 ]; then
                            echo "WARNING: target directory not found in container. Maven build likely failed."
                            exit 1
                        fi
                        docker cp ${mavenContainerName}:${containerProjectRoot}/target ./target || { echo "ERROR: Failed to copy target directory from container!"; exit 1; }
                        echo "--- Contents of copied target/classes on Jenkins host ---"
                        ls -l target/classes || { echo "ERROR: target/classes NOT found on Jenkins host after copy!"; exit 1; }
                        echo "--- Contents of target/classes on Jenkins host (detail) ---"
                        ls -l target/classes
                    """

                    // Clean up the Maven container
                    sh "docker rm ${mavenContainerName} || true" # Add || true to gracefully handle if container already removed
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                echo 'Running SonarQube analysis...'
                withCredentials([string(credentialsId: "${env.SONAR_TOKEN_CREDENTIAL_ID}", variable: 'SONAR_TOKEN')]) {
                    // Use the same consistent internal path for the SonarScanner container
                    def containerProjectRoot = "/app"

                    // DEBUG: Verify target/classes exists INSIDE the SonarScanner container
                    sh '''
                      echo "--- Checking target/classes INSIDE SonarScanner container before analysis ---"
                      docker run --rm \
                        -v "$PWD":"${containerProjectRoot}" \
                        -w "${containerProjectRoot}" \
                        sonarsource/sonar-scanner-cli \
                        ls -l target/classes || { echo "ERROR: target/classes NOT found inside SonarScanner container for analysis!"; exit 1; }

                      echo "--- Running SonarScanner CLI command now ---"
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

                    echo "--- Attempting to upload WAR file to Nexus ---"
                    curl -v -u ${NEXUS_USERNAME}:${NEXUS_PASSWORD} --upload-file "$WAR_FILE" \
                      ${NEXUS_URL}${APP_CONTEXT}/0.1-SNAPSHOT/${APP_CONTEXT}-0.1-SNAPSHOT.war || { echo "ERROR: Nexus upload failed!"; exit 1; }
                '''
            }
        }

        stage('Slack Notification') {
            steps {
                echo 'Sending Slack notification...'
                sh """
                    echo "--- Sending Slack notification ---"
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

                    echo "--- Deploying WAR to Tomcat Manager ---"
                    curl -T "$WAR_FILE" \
                      "${TOMCAT_URL}/deploy?path=/${APP_CONTEXT}&update=true" \
                      --user ${TOMCAT_USERNAME}:${TOMCAT_PASSWORD} || { echo "ERROR: Tomcat deployment failed!"; exit 1; }
                '''
            }
        }
    }
}

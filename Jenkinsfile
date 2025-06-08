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
                sh 'echo "--- Contents of src/com/room/sample/view/Test directory (Jenkins Host) ---"'
                sh 'ls -l src/com/room/sample/view/Test || echo "Test directory not found or empty." '
                sh 'echo "--- Finding all .java files in src directory (Jenkins Host) ---"'
                sh 'find src -name "*.java" || echo "No .java files found in src." '
            }
        }

        stage('Build with Maven') {
            steps {
                echo 'Building with Maven inside Docker...'
                script { // Using script block for advanced shell logic and variable handling
                    def mavenContainerName = "maven-build-${UUID.randomUUID()}"
                    def containerAppPath = "/usr/src/app" // Internal path for your project inside Maven container

                    // DEBUG: List contents inside the Maven container before build to confirm mount
                    sh """
                        echo "--- Contents inside Maven container (${containerAppPath}) before build ---"
                        docker run --rm \
                          -v "$PWD":"${containerAppPath}" \
                          -w "${containerAppPath}" \
                          maven:3.8.6-eclipse-temurin-17 \
                          ls -l "${containerAppPath}/src/com/room/sample/view" || echo "Maven container src/com/room/sample/view not found."

                        echo "--- Running Maven build ---"
                        docker run -d --name ${mavenContainerName} \
                          -v "$PWD":"${containerAppPath}" \
                          -v /var/lib/jenkins/.m2:/root/.m2 \
                          -w "${containerAppPath}" \
                          maven:3.8.6-eclipse-temurin-17 \
                          mvn clean compile package -DskipTests
                    """

                    // Wait for the Maven command inside the container to finish and stream its logs
                    sh "docker logs -f ${mavenContainerName} & PID=\$!; docker wait ${mavenContainerName}; kill \$PID"

                    // Copy the 'target' directory from container to Jenkins host workspace
                    sh """
                        echo "--- Copying target directory from Maven container to Jenkins workspace ---"
                        docker cp ${mavenContainerName}:${containerAppPath}/target ./target || { echo "ERROR: Failed to copy target directory from container!"; exit 1; }
                        echo "--- Contents of copied target/classes on Jenkins host ---"
                        ls -l target/classes || { echo "ERROR: target/classes NOT found on Jenkins host after copy!"; exit 1; }
                        echo "--- Contents of target/classes on Jenkins host (detail) ---"
                        ls -l target/classes
                    """

                    // Clean up the Maven container
                    sh "docker rm ${mavenContainerName}"
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                echo 'Running SonarQube analysis...'
                withCredentials([string(credentialsId: "${env.SONAR_TOKEN_CREDENTIAL_ID}", variable: 'SONAR_TOKEN')]) {
                    sh '''
                      # SonarScanner container will now reliably find target/classes on the Jenkins host
                      CONTAINER_WORKSPACE=/usr/src/project # Consistent internal mount point for SonarScanner

                      // DEBUG: Verify target/classes exists INSIDE the SonarScanner container
                      echo "--- Checking target/classes INSIDE SonarScanner container before analysis ---"
                      docker run --rm \
                        -v "$PWD":"${CONTAINER_WORKSPACE}" \
                        -w "${CONTAINER_WORKSPACE}" \
                        sonarsource/sonar-scanner-cli \
                        ls -l target/classes || { echo "ERROR: target/classes NOT found inside SonarScanner container for analysis!"; exit 1; }

                      echo "--- Running SonarScanner CLI command now ---"
                      docker run --rm \
                        -e SONAR_HOST_URL=${SONAR_HOST} \
                        -e SONAR_TOKEN=${SONAR_TOKEN} \
                        -v "$PWD":"${CONTAINER_WORKSPACE}" \
                        -w "${CONTAINER_WORKSPACE}" \
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

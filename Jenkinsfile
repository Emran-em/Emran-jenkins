pipeline {
    agent any

    environment {
        SONAR_HOST = 'http://52.23.219.98:9000'
        SONAR_TOKEN_CREDENTIAL_ID = 'sonar'
        // Corrected NEXUS_URL to a more standard path, removed '/in/'
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
                sh '''
                    # Mount the Jenkins workspace ($PWD) into the Maven container at /usr/src/mymaven.
                    # This ensures 'target/classes' and 'target/*.war' are created directly in $PWD on the Jenkins host.
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
                      # Mount the entire Jenkins workspace ($PWD) into the SonarScanner container.
                      # This makes 'target/classes' visible at /usr/src/mymaven/target/classes
                      # SonarScanner will run from /usr/src/mymaven, so relative paths work.
                      docker run --rm \
                        -e SONAR_HOST_URL=${SONAR_HOST} \
                        -e SONAR_TOKEN=${SONAR_TOKEN} \
                        -v "$PWD":/usr/src/mymaven \
                        -w /usr/src/mymaven \
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
                    # WAR_FILE is expected in $PWD/target/, which is correct if Maven built there.
                    WAR_FILE=$(find target -name "*.war" | head -n 1)

                    if [ ! -f "$WAR_FILE" ]; then
                      echo "WAR file not found, build may have failed."
                      exit 1
                    fi

                    # Ensure your Nexus repository is configured to allow direct curl uploads to this path.
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
                      --data '{"text":"✅ *Build SUCCESSFUL* for *${APP_CONTEXT}* on *${GIT_BRANCH}* branch! 🚀"}' \
                      ${SLACK_WEBHOOK_URL}
                """
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                echo 'Deploying WAR to Tomcat...'
                sh '''
                    # WAR_FILE is expected in $PWD/target/, which is correct.
                    WAR_FILE=$(find target -name "*.war" | head -n 1)

                    curl -T "$WAR_FILE" \
                      "${TOMCAT_URL}/deploy?path=/${APP_CONTEXT}&update=true" \
                      --user ${TOMCAT_USERNAME}:${TOMCAT_PASSWORD}
                '''
            }
        }
    }
}

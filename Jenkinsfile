pipeline {
    agent any

    tools {
        maven 'MVN_HOME'
    }

    environment {
        // Nexus details
        NEXUS_VERSION     = "nexus3"
        NEXUS_PROTOCOL    = "http"
        NEXUS_URL         = "3.83.214.6:8081"
        NEXUS_REPOSITORY  = "Emran-NX-repo"
        NEXUS_CREDENTIAL_ID = "nexus-credentials"

        // SonarQube scanner tool
        SCANNER_HOME = tool 'sonar_scanner'

        // Slack details
        SLACK_CHANNEL = "#jenkins-integration"
    }

    stages {
        stage("Clone Code") {
            steps {
                git branch: 'master', url: 'https://github.com/Emran-em/Emran-jenkins.git'
            }
        }

        stage("Maven Build") {
            steps {
                sh 'mvn -Dmaven.test.failure.ignore=true clean install'
            }
        }

        stage("SonarQube Analysis") {
            steps {
                withSonarQubeEnv('sonarqube-server') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectKey=emran-jenkins \
                        -Dsonar.projectName="Emran Jenkins App" \
                        -Dsonar.projectVersion=1.0 \
                        -Dsonar.sources=src/main/java \
                        -Dsonar.java.binaries=target/classes '''
                }
            }
        }

        stage("Publish to Nexus") {
            steps {
                script {
                    pom = readMavenPom file: "pom.xml"
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}")
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path}"
                    artifactPath = filesByGlob[0].path
                    artifactExists = fileExists artifactPath

                    if (artifactExists) {
                        nexusArtifactUploader(
                            nexusVersion: NEXUS_VERSION,
                            protocol: NEXUS_PROTOCOL,
                            nexusUrl: NEXUS_URL,
                            groupId: pom.groupId,
                            version: pom.version,
                            repository: NEXUS_REPOSITORY,
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                [artifactId: pom.artifactId, classifier: '', file: artifactPath, type: pom.packaging],
                                [artifactId: pom.artifactId, classifier: '', file: "pom.xml", type: "pom"]
                            ]
                        )
                    } else {
                        error "*** File: ${artifactPath}, could not be found"
                    }
                }
            }
        }

        stage("Deploy to Tomcat") {
            steps {
                withCredentials([usernamePassword(credentialsId: 'deployer', usernameVariable: 'TOMCAT_USER', passwordVariable: 'TOMCAT_PASS')]) {
                    script {
                        // WAR file built by Maven
                        def warFile = sh(script: "ls target/*.war | head -n 1", returnStdout: true).trim()

                        echo "Deploying ${warFile} to Tomcat at context path /emran-app ..."

                        sh """
                            curl -u $TOMCAT_USER:$TOMCAT_PASS \
                                 -T ${warFile} \
                                 "http://54.221.151.214:8080/manager/text/deploy?path=/emran-app&update=true"
                        """
                    }
                }
            }
        }

        stage("Slack Notification") {
            steps {
                withCredentials([string(credentialsId: 'slack_notification', variable: 'SLACK_TOKEN')]) {
                    slackSend(
                        channel: "${SLACK_CHANNEL}",
                        color: "#36a64f",
                        message: "✅ Jenkins Declarative Pipeline for *Emran Jenkins App* deployed successfully to Tomcat! Job: ${env.JOB_NAME} [${env.BUILD_NUMBER}]",
                        token: "${SLACK_TOKEN}"
                    )
                }
            }
        }
    }
}

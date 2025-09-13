pipeline {
    agent any

    tools {
        maven 'MVN_HOME'
    }

    environment {
        // Nexus details
        NEXUS_VERSION       = "nexus3"
        NEXUS_PROTOCOL      = "http"
        NEXUS_URL           = "18.206.235.190:8081"
        NEXUS_REPOSITORY    = "Emran-NX-repo"
        NEXUS_CREDENTIAL_ID = "NX"

        // SonarQube scanner tool
        SCANNER_HOME = tool 'sonar_scanner'

        // Slack details
        SLACK_CHANNEL = "#jenkins-integration"
        
        // Manual Java 17 path
        JAVA_HOME = "/usr/lib/jvm/java-17-amazon-corretto.x86_64"
        PATH = "${JAVA_HOME}/bin:${PATH}"
    }

    stages {
        stage("Setup JDK") {
            steps {
                script {
                    sh 'java -version'
                    sh 'javac -version'
                    sh 'echo "JAVA_HOME is set to: $JAVA_HOME"'
                }
            }
        }

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
        withSonarQubeEnv('sonar_scanner') {
            sh """
                export JAVA_HOME=/usr/lib/jvm/java-17-amazon-corretto.x86_64
                /opt/sonar-scanner/bin/sonar-scanner \
                    -Dsonar.projectKey=emran-jenkins \
                    -Dsonar.projectName='Emran Jenkins App' \
                    -Dsonar.projectVersion=2.0 \
                    -Dsonar.sources=src/
                    -Dsonar.sources=src/main/java \
                    -Dsonar.java.binaries=target/classes \
                    -Dsonar.java.home=/usr/lib/jvm/java-17-amazon-corretto.x86_64
                    -Dsonar.java.binaries=src/
            """
                }
            }
        }

        stage("Publish to Nexus") {
            steps {
                script {
                    def pom = readMavenPom file: "pom.xml"
                    def filesByGlob = findFiles(glob: "target/*.${pom.packaging}")
                    if (filesByGlob.length == 0) {
                        error "*** File: target/*.${pom.packaging} not found"
                    }
                    def artifactPath = filesByGlob[0].path
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
                }
            }
        }

        stage("Deploy to Tomcat") {
            steps {
                withCredentials([usernamePassword(credentialsId: 'deployer', usernameVariable: 'TOMCAT_USER', passwordVariable: 'TOMCAT_PASS')]) {
                    script {
                        def warFile = sh(script: "ls target/*.war | head -n 1", returnStdout: true).trim()
                        echo "Deploying ${warFile} to Tomcat at /emran-app ..."
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

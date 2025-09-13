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
        
        // Java 17 path
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
                withSonarQubeEnv('sonarqube-server') {
                    sh """
                        ${SCANNER_HOME}/bin/sonar-scanner \
                          -Dsonar.projectKey=emran-jenkins \
                          -Dsonar.sources=src \
                          -Dsonar.java.binaries=target \
                          -Dsonar.host.url=http://34.229.178.233:9001 \
                          -Dsonar.login=sonarqube
                    """
                }
            }
        }

        stage("Publish to Nexus") {
    steps {
        script {
            def artifactId = sh(script: "mvn help:evaluate -Dexpression=project.artifactId -q -DforceStdout", returnStdout: true).trim()
            def groupId    = sh(script: "mvn help:evaluate -Dexpression=project.groupId -q -DforceStdout", returnStdout: true).trim()
            def version    = sh(script: "mvn help:evaluate -Dexpression=project.version -q -DforceStdout", returnStdout: true).trim()
            def packaging  = sh(script: "mvn help:evaluate -Dexpression=project.packaging -q -DforceStdout", returnStdout: true).trim()

            def filesByGlob = findFiles(glob: "target/*.${packaging}")
            if (filesByGlob.length == 0) {
                error "No ${packaging} file found in target directory!"
            }
            def artifactPath = filesByGlob[0].path

            echo "*** File: ${artifactPath}, group: ${groupId}, packaging: ${packaging}, version: ${version}"

            nexusArtifactUploader(
                nexusVersion: NEXUS_VERSION,
                protocol: 'http',
                nexusUrl: '18.206.235.190:8081',
                groupId: 'com.emran',
                version: '1.0-SNAPSHOT',
                repository: 'Emran-NX-repo',
                credentialsId: 'NX',
                artifacts: [
                    [artifactId: artifactId, classifier: '', file: artifactPath, type: packaging],
                    [artifactId: artifactId, classifier: '', file: "pom.xml", type: "pom"]
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

pipeline {
    agent any
    tools {
        maven 'Maven' // Adjust to match your Jenkins Maven tool name
        jdk 'JDK8'    // Adjust to match your Jenkins JDK tool name
    }
    stages {
        stage('clone code') {
            steps {
                script {
                    git url: 'https://github.com/betawins/sabear_simplecutomerapp.git', branch: 'master'
                }
            }
        }
        stage('mvn build') {
            steps {
                script {
                    sh 'mvn -Dmaven.test.failure.ignore=true clean install'
                }
            }
        }
        stage('SonarCloud') {
            steps {
                withSonarQubeEnv('sonarqube_server') {
                    sh '/var/lib/jenkins/tools/hudson.plugins.sonar.SonarRunnerInstallation/sonar_scanner/bin/sonar-scanner \
                        -Dsonar.projectKey=Ncodeit \
                        -Dsonar.projectName=Ncodeit \
                        -Dsonar.projectVersion=2.0 \
                        -Dsonar.sources=/var/lib/jenkins/workspace/Simplecutomerapp/src/ \
                        -Dsonar.binaries=target/classes/com/visualpathit/account/controller/ \
                        -Dsonar.junit.reportsPath=target/surefire-reports \
                        -Dsonar.jacoco.reportPath=target/jacoco.exec \
                        -Dsonar.java.binaries=src/com/room/sample'
                }
            }
        }
        stage('publish to nexus') {
            steps {
                script {
                    def pomXml = readFile('pom.xml')
                    def xml = new XmlSlurper().parseText(pomXml)
                    def packaging = xml.packaging.text() ?: 'jar' // Default to 'jar' if not specified
                    // Example Nexus publishing (adjust based on your setup)
                    nexusPublisher(
                        nexusInstanceId: 'nexus', // Replace with your Nexus instance ID
                        nexusRepositoryId: 'maven-snapshots', // Replace with your repository ID
                        packages: [[
                            $class: 'MavenPackage',
                            mavenCoordinate: [
                                groupId: 'com.javatpoint',
                                artifactId: 'SimpleCustomerApp',
                                version: env.BUILD_NUMBER + '-SNAPSHOT',
                                packaging: packaging
                            ],
                            mavenAssetList: [[
                                classifier: '',
                                extension: packaging,
                                filePath: "target/SimpleCustomerApp-${env.BUILD_NUMBER}-SNAPSHOT.${packaging}"
                            ]]
                        ]]
                    )
                }
            }
        }
    }
}

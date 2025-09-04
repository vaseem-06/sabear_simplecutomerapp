pipeline {
    agent any

    environment {
        MAVEN_HOME = '/opt/apache-maven-3.9.11'
        JAVA_HOME  = '/usr/lib/jvm/java-17-amazon-corretto.x86_64'
        PATH = "${MAVEN_HOME}/bin:${JAVA_HOME}/bin:${env.PATH}"

        // Nexus details
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "98.82.8.146:8081"
        NEXUS_REPOSITORY = "sonarqube"
        NEXUS_CREDENTIAL_ID = "nexus_server"
    }

    stages {
        stage('Checkout SCM') {
            steps {
                git branch: 'master', url: 'https://github.com/vaseem-06/sabear_simplecutomerapp.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Deploy to Nexus') {
            steps {
                script {
                    pom = readMavenPom file: "pom.xml"
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}")

                    if (filesByGlob.length > 0) {
                        def artifactPath = filesByGlob[0].path
                        echo "*** Uploading: ${artifactPath}"

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
                        error "Artifact not found in target/"
                    }
                }
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                script {
                    def deployFailed = false
                    withCredentials([usernamePassword(credentialsId: 'tomcat_credentials',
                                                     usernameVariable: 'TOMCAT_USER',
                                                     passwordVariable: 'TOMCAT_PASS')]) {
                        try {
                            sh """
                            curl -u $TOMCAT_USER:$TOMCAT_PASS \
                            -T target/*.war \
                            http://52.87.164.57:8080/manager/text/deploy?path=/hiring&update=true
                            """
                        } catch (err) {
                            deployFailed = true
                            echo "Deployment failed!"
                        }
                    }

                    if (deployFailed) {
                        echo "Rolling back to previous WAR..."
                        withCredentials([usernamePassword(credentialsId: 'tomcat_credentials',
                                                          usernameVariable: 'TOMCAT_USER',
                                                          passwordVariable: 'TOMCAT_PASS')]) {
                            sh """
                            curl -u $TOMCAT_USER:$TOMCAT_PASS \
                            -T previous.war \
                            http://52.87.164.57:8080/manager/text/deploy?path=/hiring&update=true
                            """
                        }
                        error("Deployment failed and rollback executed.")
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline finished successfully!'
            slackSend(channel: '#jenkins-integration',
                      color: 'good',
                      tokenCredentialId: 'slack_integrations',
                      message: "SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}")
        }
        failure {
            echo 'Pipeline failed!'
            slackSend(channel: '#jenkins-integration',
                      color: 'danger',
                      tokenCredentialId: 'slack_integrations',
                      message: "FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}")
        }
    }
}

pipeline {
    agent any

    tools {
        maven "MVN_HOME"
    }

    environment {
        NEXUS_VERSION      = "nexus3"
        NEXUS_PROTOCOL     = "http"
        NEXUS_URL          = "98.82.8.146:8081"
        NEXUS_REPOSITORY   = "maven-releases"
        NEXUS_CREDENTIAL_ID= "nexus_server"
    }

    stages {
        stage("Checkout") {
            steps {
                git branch: 'master', url: 'https://github.com/vaseem-06/hiring-app-1.git'
            }
        }

        stage("Build & Test") {
            steps {
                sh 'mvn clean package -Dmaven.test.failure.ignore=true'
            }
        }

        stage("Publish to Nexus") {
            steps {
                script {
                    pom = readMavenPom file: "pom.xml"
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}")
                    artifactPath = filesByGlob[0].path

                    nexusArtifactUploader(
                        nexusVersion: NEXUS_VERSION,
                        protocol: NEXUS_PROTOCOL,
                        nexusUrl: NEXUS_URL,
                        groupId: pom.groupId,
                        version: pom.version,
                        repository: NEXUS_REPOSITORY,
                        credentialsId: NEXUS_CREDENTIAL_ID,
                        artifacts: [
                            [artifactId: pom.artifactId,
                             classifier: '',
                             file: artifactPath,
                             type: pom.packaging],
                            [artifactId: pom.artifactId,
                             classifier: '',
                             file: "pom.xml",
                             type: "pom"]
                        ]
                    )
                }
            }
        }

        stage("Deploy to Tomcat") {
            steps {
                withCredentials([usernamePassword(credentialsId: 'tomcat_credentials',
                                                 usernameVariable: 'TOMCAT_USER',
                                                 passwordVariable: 'TOMCAT_PASS')]) {
                    sh """
                        curl -u $TOMCAT_USER:$TOMCAT_PASS \
                        -T target/hiring-0.1.war \
                        http://52.87.164.57:8080/manager/text/deploy?path=/hiring&update=true
                    """
                }
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline finished successfully!"
            withCredentials([string(credentialsId: 'slack_integrations', variable: 'SLACK_TOKEN')]) {
                slackSend(channel: '#build-notifications',
                          color: 'good',
                          token: SLACK_TOKEN,
                          message: "✅ Build & Deploy Succeeded: ${env.JOB_NAME} #${env.BUILD_NUMBER}")
            }
        }
        failure {
            echo "❌ Pipeline failed!"
            withCredentials([string(credentialsId: 'slack_integrations', variable: 'SLACK_TOKEN')]) {
                slackSend(channel: '#build-notifications',
                          color: 'danger',
                          token: SLACK_TOKEN,
                          message: "❌ Build/Deploy Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}")
            }
        }
    }
}

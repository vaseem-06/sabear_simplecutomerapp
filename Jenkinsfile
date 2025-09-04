pipeline {
    agent {
        label "master"
    }
    tools {
        maven "MVN_HOME"
    }
    environment {
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "98.82.8.146:8081"
        NEXUS_REPOSITORY = "maven-releases"   // <-- apne Nexus repo ka sahi naam dalna
        NEXUS_CREDENTIAL_ID = "nexus_server"
    }
    stages {
        stage("Clone Code") {
            steps {
                script {
                    git 'https://github.com/vaseem-06/sabear_simplecutomerapp.git'
                }
            }
        }
        stage("Maven Build") {
            steps {
                script {
                    sh 'mvn -Dmaven.test.failure.ignore=true install'
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
                        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}"
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
    }
}

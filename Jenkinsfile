pipeline {
    agent any
    environment {
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "35.175.232.176:8081"
        NEXUS_CREDENTIAL_ID = "nexus-credentials"
    }
    stages {
        stage("Checkout") {
            steps {
                git branch: 'main', 
                    url: 'https://github.com/Naresh240/springboot-with-jenkins-CICD.git'
            }
        }
        stage("Build_Artifact") {
            steps {
                sh "mvn clean package"
            }
        }
        stage("Code_Analysis") {
            steps {
                withSonarQubeEnv(installationName: 'sonarqube', credentialsId: 'sonar-token') {
                    sh "${ tool ("sonar-scanner")}/sonar-scanner \
                        -Dsonar.projectKey=hellospringboot \
                        -Dsonar.projectName=hellospringboot \
                        -Dsonar.sourceEncoding=UTF-8 \
                        -Dsonar.sources=src"
                }
            }
        }
        stage("Quality_Gate"){
            steps {
                script {
                    timeout(time: 3, unit: 'MINUTES') {
                        echo "Initializing quality gates..."
                        sh 'sleep 10'
                        def result = waitForQualityGate()
                        if (result.status != 'OK') {
                            error "Pipeline aborted due to quality gate failure: ${result.status}"
                        } else {
                            echo "Quality gate passed with result: ${result.status}"
                        }
                    }
                }
            }
        }
        stage("Publish to Nexus Repository Manager") {
            steps {
                script {
                    pom = readMavenPom file: "pom.xml"
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}")
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                    artifactPath = filesByGlob[0].path
                    artifactExists = fileExists artifactPath

                    if (artifactExists) {
                        def isSnapshot = pom.version.contains("-SNAPSHOT")
                        def nexusRepository = isSnapshot ? "maven-snapshots" : "maven-releases"

                        echo "* File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}, repository: ${nexusRepository}"

                        nexusArtifactUploader(
                            nexusVersion: NEXUS_VERSION,
                            protocol: NEXUS_PROTOCOL,
                            nexusUrl: NEXUS_URL,
                            groupId: pom.groupId,
                            version: pom.version,
                            repository: nexusRepository,
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
                    } else {
                        error "* File: ${artifactPath}, could not be found"
                    }
                }
            }
        }
    }
}
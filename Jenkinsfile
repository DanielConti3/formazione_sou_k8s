pipeline {
    environment {
        imagename = "contid/track2"
        registryCredential = 'DockerHub'
    }
    agent any
    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Cloning Git') {
            steps {
                script {
                    // Clona il repository senza specificare un branch fisso
                    checkout scm
                    // Ottieni l'ultimo tag Git disponibile
                    env.GIT_TAG = sh(script: 'git describe --tags --abbrev=0', returnStdout: true).trim()
                    // Ottieni il nome del branch
                    env.BRANCH_NAME = env.GIT_BRANCH.replaceAll('origin/', '')
                    echo "Cloned Branch: ${env.BRANCH_NAME}"
                    echo "Git Tag: ${env.GIT_TAG}"
                }
            }
        }
        stage('Build image') {
            steps {
                script {
                    def dockerArgs = buildImage()
                    def dockerfilePath = "${dockerArgs.dockerfileDir}/${dockerArgs.dockerfileName}"
                    customImage = docker.build("${imagename}:${env.GIT_TAG}", "-f ${dockerfilePath} ${dockerArgs.dockerfileDir}")
                }
            }
        }
        stage('Debug Info') {
            steps {
                script {
                    echo "Branch Name: ${env.BRANCH_NAME}"
                    echo "Git Commit: ${env.GIT_COMMIT}"
                    echo "Git Tag: ${env.GIT_TAG}"
                }
            }
        }
        stage('Deploy Image') {
            steps {
                script {
                    docker.withRegistry('', env.registryCredential) {
                        def tag = ""
                        def additionalTag = ""
                        if (env.GIT_TAG && env.GIT_TAG != "") {
                            tag = env.GIT_TAG
                            additionalTag = 'latest'
                        } else if (env.BRANCH_NAME == 'main') {
                            tag = 'latest'
                        } else if (env.BRANCH_NAME == 'secondary') {
                            tag = "secondary-${env.GIT_COMMIT}"
                        } else {
                            tag = "${env.BRANCH_NAME}-${env.GIT_COMMIT}"
                        }
                        customImage.push(tag)
                        if (additionalTag) {
                            customImage.push(additionalTag)
                        }
                    }
                }
            }
        }
        stage('Remove Unused docker image') {
            steps {
                script {
                    def tag = ""
                    def additionalTag = ""
                    if (env.GIT_TAG && env.GIT_TAG != "") {
                        tag = env.GIT_TAG
                        additionalTag = 'latest'
                    } else if (env.BRANCH_NAME == 'main') {
                        tag = 'latest'
                    } else if (env.BRANCH_NAME == 'secondary') {
                        tag = "secondary-${env.GIT_COMMIT}"
                    } else {
                        tag = "${env.BRANCH_NAME}-${env.GIT_COMMIT}"
                    }
                    def commitImage = "${imagename}:${env.GIT_COMMIT}"
                    def tagImage = "${imagename}:${tag}"
                    def additionalTagImage = "${imagename}:${additionalTag}"

                    // Check and remove images if they exist
                    if (sh(script: "docker images -q ${commitImage}", returnStdout: true).trim()) {
                        sh "docker rmi ${commitImage}"
                    }
                    if (sh(script: "docker images -q ${tagImage}", returnStdout: true).trim()) {
                        sh "docker rmi ${tagImage}"
                    }
                    if (sh(script: "docker images -q ${additionalTagImage}", returnStdout: true).trim()) {
                        sh "docker rmi ${additionalTagImage}"
                    }
                }
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}

@NonCPS
def buildImage() {
    def defaults = [
        registryUrl: 'https://hub.docker.com/repository/docker/contid/track2/general',
        //dockerfileDir: "/var/jenkins_home/workspace/flask-app-example-build_main/flask-app",
        dockerfileDir: "${WORKSPACE}/flask-app",
        dockerfileName: "Dockerfile",
        buildArgs: ""
    ]
    return defaults
}

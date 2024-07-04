pipeline {
    environment {
        imagename = "contid/track2"
        registryCredential = 'DockerHub'
        //customImage = ""
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
                }
            }
        }
        stage('Build image') {
            steps {
                script {
                    dockerArgs = buildImage()
                    dockerArgs = "${dockerArgs.buildArgs} -f ${dockerArgs.dockerfileName} ${WORKSPACE}/flask-app/"
                    docker.build(contid/track2, dockerArgs)
                    //def customImage = sh "docker build . -t contid/track2:1.0.1 -f ${WORKSPACE}/flask-app/Dockerfile"
                }
            }
        }
        stage('Deploy Image') {
            steps {
                script {
                    docker.withRegistry(env.registryCredential) {
                        def tag = ""
                        def additionalTag = ""
                        def customImage = sh "docker build . -t contid/track2:1.0.1 -f ${WORKSPACE}/flask-app/Dockerfile"
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
                        //docker.withRegistry(env.registryCredential) {
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
                    sh "docker rmi ${imagename}:${env.GIT_COMMIT}"
                    sh "docker rmi ${imagename}:${tag}"
                    if (additionalTag) {
                        sh "docker rmi ${imagename}:${additionalTag}"
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
    defaults = [
        registryUrl: 'https://hub.docker.com/repository/docker/contid/track2/general',
        dockerfileDir: "/var/jenkins_home/workspace/flask-app-example-build_main/flask-app",
        dockerfileName: "Dockerfile",
        buildArgs: "",
    ]
    //args = defaults + args
    args = defaults
    return args
}

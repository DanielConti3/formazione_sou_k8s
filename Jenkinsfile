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
                }
            }
        }
        stage('Building Docker image') {
            steps {
                docker build {
                    dockerfile {
                        filename 'Dockerfile'
                        dir '/Jenkins/'
                    }
                }
            }
        }
        stage('Deploy Image') {
            steps {
                script {
                  //  def tag = ""
                  //  def additionalTag = ""
                  //  if (env.GIT_TAG && env.GIT_TAG != "") {
                      //  tag = env.GIT_TAG
                      //  additionalTag = 'latest'
                  //  } else if (env.BRANCH_NAME == 'main') {
                      //  tag = 'latest'
                  //  } else if (env.BRANCH_NAME == 'secondary') {
                   //     tag = "secondary-${env.GIT_COMMIT}"
                  //  } else {
                   //     tag = "${env.BRANCH_NAME}-${env.GIT_COMMIT}"
                  //  }
                    docker.withRegistry('', registryCredential) {
                        docker.build(imageName: "${imagename}:${tag}").push()
                        if (additionalTag) {
                            dockerImage.build(imageName: "${imagename}:${additionalTag}").push()
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

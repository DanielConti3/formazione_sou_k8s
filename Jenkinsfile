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
        pushLatest: true
    ]
    args = defaults + args
    //args = defaults
    docker.withRegistry(args.registryUrl) {
        def customImage = sh "docker build . -t contid/track2:1.0.1 -f ${WORKSPACE}/flask-app/Dockerfile"
        image.push(args.buildTag)
        if(args.pushLatest) {
            image.push("latest")
            sh "docker rmi --force ${args.image}:latest"
        }
        sh "docker rmi --force ${args.image}:${args.buildTag}"
    return args
  }
}

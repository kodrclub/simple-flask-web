pipeline {
    agent any
    triggers {
        pollSCM('* * * * */1')
    }
     options {
        disableConcurrentBuilds()
    }
    environment {
        registry = "kodrclub/simple-flask-web"
        registryCredential = 'Docker'
        apiServer = "https://kubernetes.docker.internal:6443"
        devNamespace = "default"
        minikubeCredential = 'minikube-auth-token'
        imageTag = "${env.GIT_BRANCH + '_' + env.BUILD_NUMBER}"
    }
    stages {
        stage('Test') {
            steps {
                echo 'Testing..'
            }
        }
        stage('Build image') {
            steps {
                script {
                    dockerImage = docker.build(registry + ":$imageTag","--network host .")
                }
            }
        }
        stage('Upload to registry') {
            steps{
                script {
                    docker.withRegistry( '', registryCredential ) {
                        dockerImage.push()
                    }
                }
            }
        }
        stage('Deploy to K8s') {
            steps{
                withKubeConfig([credentialsId: minikubeCredential,
                                serverUrl: apiServer,
                                namespace: devNamespace
                               ]) {
                    sh 'kubectl set image deployment/flask flask="$registry:$imageTag" --record'
                }
            }
        }
    }
}
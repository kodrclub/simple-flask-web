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
        stage('Update deployment file') {
            when {
                expression { env.GIT_BRANCH == 'develop' }
            }
            steps{
                script {
                    withCredentials([usernamePassword(credentialsId: 'Git-Encoded', usernameVariable: 'username', passwordVariable: 'password')]){
                        sh "rm -rf simple-flask-web-deployment"
                        sh "git clone https://$username:$password@github.com/kodrclub/simple-flask-web-deployment.git"
                        dir("simple-flask-web-deployment") {
                            sh "echo \"spec:\n  template:\n    spec:\n      containers:\n        - name: django\n          image: ${registry}:$imageTag\" > patch.yaml"
                            sh "kubectl patch --local -o yaml -f django-deployment.yaml -p \"\$(cat patch.yaml)\" > new-deploy.yaml"
                            sh "mv new-deploy.yaml django-deployment.yaml"
                            sh "rm patch.yaml"
                            sh "git add ."
                            sh "git commit -m\"Patched deployment for $imageTag\""
                            sh "git push https://$username:$password@github.com/kodrclub/simple-flask-web-deployment.git"
                        }
                    }
                }
            }
        }
        stage('Deploy to K8s') {
            when {
                expression { env.GIT_BRANCH == 'develop' }
            }
            steps{
                withKubeConfig([credentialsId: minikubeCredential,
                                serverUrl: apiServer,
                                namespace: devNamespace
                               ]) {
                    sh 'kubectl apply -f simple-flask-web-deployment/django-deployment.yaml'
                }
            }
        }
    }
}
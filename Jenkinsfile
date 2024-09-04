pipeline {
    agent any

    environment {
        IMAGE_NAME = "geraldakenji/guess-game:v.0.0-${env.BUILD_NUMBER}-lite"
        DOCKERHUB_CREDENTIALS = credentials('d0c52560-0c6e-4ac6-9953-550e905586ea')
    }

    stages {
        stage("Clean Workspace") {
            steps {
                cleanWs()
            }
        }
        stage("Git Checkout") {
            steps {
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/GeraldAkenji/guessgame.git']])
            }
        }
        stage("Docker Login") {
            steps {
                script {
                    sh "echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin"
                }
            }
        }
        stage("Build Docker Image") {
            steps {
                script {
                    sh "docker build -t $IMAGE_NAME ."
                }
            }
        }
        stage("Push to DockerHub") {
            steps {
                script {
                    sh "docker push $IMAGE_NAME"
                }
            }
        }
        stage("Deploy to K8S") {
            steps {
                script {
                    dir('./k8s') {
                        withKubeCredentials(kubectlCredentials: [[caCertificate: '', clusterName: '', contextName: '', credentialsId: '88a9f11c-11e5-4bdb-b3bd-f63dba417648', namespace: '', serverUrl: '']]) {
                            sh "sed -i 's|IMAGE_NAME|$IMAGE_NAME|g' deploy.yaml"
                            sh "kubectl apply -f ."
                            echo "Deployed.. Check the namespace"
                        }
                    }
                }
            }
        }
    }
}
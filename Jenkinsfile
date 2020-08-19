pipeline {
    agent any
        environment {
        PROJECT_ID = 'playground-s-11-513d5d1c'
        CLUSTER_NAME = 'ks'
        LOCATION = 'us-central1-c'
        CREDENTIALS_ID = 'playground-s-11-513d5d1c'
        DOCKER_IMAGE_NAME = "shuvamoy008/train-schedule-test"
    }
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    app = docker.build(DOCKER_IMAGE_NAME)
                    app.inside {
                        sh 'echo Hello, World!'
                    }
                }
            }
        }
        stage('Push Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        stage('Deploy to GKE') {
            steps{
                sh "sed -i 's/train-schedule-test:latest/train-schedule-test:${env.BUILD_NUMBER}/g' train-schedule-kube.yaml"
                step([$class: 'KubernetesEngineBuilder', projectId: env.PROJECT_ID, clusterName: env.CLUSTER_NAME, location: env.LOCATION, manifestPattern: 'train-schedule-kube.yaml', credentialsId: env.CREDENTIALS_ID, verifyDeployments: true])
            }
        }
        stage('Deploy strimzi') {
            steps{
               sh "helm repo add strimzi https://strimzi.io/charts/"
               sh "helm install strimzi/strimzi-kafka-operator --generate-name"
            }
        }
                
    }
}

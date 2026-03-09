pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "frankyfeukeu/exam_jenkins"
        DOCKER_REGISTRY = "https://registry.hub.docker.com"
    }

    stages {

        stage('Clone') {
            steps {
                git 'https://github.com/riquelos/Jenkins_devops_exams.git'
            }
        }

        stage('Build Docker') {
            steps {
                sh 'docker build -t $IMAGE:$BUILD_NUMBER .'
            }
        }

        stage('Push DockerHub') {
            steps {
                withDockerRegistry([credentialsId: 'dockerhub']) {
                    sh 'docker push $IMAGE:$BUILD_NUMBER'
                }
            }
        }

        stage('Deploy DEV') {
            steps {
                sh 'helm upgrade --install app ./charts -n dev'
            }
        }

        stage('Deploy QA') {
            steps {
                sh 'helm upgrade --install app ./charts -n qa'
            }
        }

        stage('Deploy STAGING') {
            steps {
                sh 'helm upgrade --install app ./charts -n staging'
            }
        }

        stage('Deploy PROD') {
            when {
                branch 'master'
            }

            steps {
                input message: "Deploy to production?"
                sh 'helm upgrade --install app ./charts -n prod'
            }
        }
    }
}

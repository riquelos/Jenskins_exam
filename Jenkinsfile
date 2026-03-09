pipeline {
    agent any

    environment {
        DOCKER_REGISTRY = "https://registry.hub.docker.com"
        DOCKERHUB_REPO = "frankyfeukeu/exam_jenkins"
    }

    stages {

        stage('Clone') {
            steps {
                git 'https://github.com/riquelos/Jenskins_exam.git'
            }
        }

        stage('Build cast-service') {
            steps {
                sh """
                    docker build \
                        -t ${DOCKERHUB_REPO}-cast:${BUILD_NUMBER} \
                        -f Jenkins_devops_exams/cast-service/Dockerfile \
                        Jenkins_devops_exams/cast-service
                """
            }
        }

        stage('Build movie-service') {
            steps {
                sh """
                    docker build \
                        -t ${DOCKERHUB_REPO}-movie:${BUILD_NUMBER} \
                        -f Jenkins_devops_exams/movie-service/Dockerfile \
                        Jenkins_devops_exams/movie-service
                """
            }
        }

        stage('Push DockerHub') {
            steps {
                withDockerRegistry([credentialsId: 'dockerhub']) {
                    sh "docker push ${DOCKERHUB_REPO}-cast:${BUILD_NUMBER}"
                    sh "docker push ${DOCKERHUB_REPO}-movie:${BUILD_NUMBER}"
                }
            }
        }

        stage('Deploy DEV') {
            steps {
                sh """
                    helm upgrade --install app ./charts -n dev \
                        --set cast.image=${DOCKERHUB_REPO}-cast:${BUILD_NUMBER} \
                        --set movie.image=${DOCKERHUB_REPO}-movie:${BUILD_NUMBER}
                """
            }
        }

        stage('Deploy QA') {
            steps {
                sh """
                    helm upgrade --install app ./charts -n qa \
                        --set cast.image=${DOCKERHUB_REPO}-cast:${BUILD_NUMBER} \
                        --set movie.image=${DOCKERHUB_REPO}-movie:${BUILD_NUMBER}
                """
            }
        }

        stage('Deploy STAGING') {
            steps {
                sh """
                    helm upgrade --install app ./charts -n staging \
                        --set cast.image=${DOCKERHUB_REPO}-cast:${BUILD_NUMBER} \
                        --set movie.image=${DOCKERHUB_REPO}-movie:${BUILD_NUMBER}
                """
            }
        }

        stage('Deploy PROD') {
            when {
                branch 'master'
            }
            steps {
                input message: "Deploy to production?"
                sh """
                    helm upgrade --install app ./charts -n prod \
                        --set cast.image=${DOCKERHUB_REPO}-cast:${BUILD_NUMBER} \
                        --set movie.image=${DOCKERHUB_REPO}-movie:${BUILD_NUMBER}
                """
            }
        }
    }
}

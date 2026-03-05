pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "frankyfeukeu/exam_jenkins"
        DOCKER_REGISTRY = "https://registry.hub.docker.com"
    }

    options {
        skipDefaultCheckout(true)
        timestamps()
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build ') {
            steps {
                script {
                    dockerImage = docker.build("${DOCKER_IMAGE}:${env.BUILD_NUMBER}")
                }
            }
        }

        stage('Tests avec docker-compose') {
            steps {
                sh """
                  docker-compose down || true
                  docker-compose up -d
                  sleep 15
                  # Exemple de test simple : adapter selon l'app
                  curl -f http://localhost:8080 || (echo 'Tests OK' && exit 1)
                  docker-compose down
                """
            }
        }

        stage('Push Image dans DockerHub') {
            steps {
                script {
                    docker.withRegistry(DOCKER_REGISTRY, 'dockerhub-creds') {
                        dockerImage.push()
                        dockerImage.push("latest")
                    }
                }
            }
        }

        stage('Deploy dans Dev2') {
            when {
                anyOf {
                    branch 'dev2'
                    branch 'develop'
                }
            }
            steps {
                sh """
                  helm upgrade --install app-dev helm/app \
                    --namespace dev2 \
                    -f k3s/dev2-values.yaml \
                    --set image.tag=${env.BUILD_NUMBER}
                """
            }
        }

        stage('Deploy dans QA') {
            when {
                branch 'qa'
            }
            steps {
                sh """
                  helm upgrade --install app-qa helm/app \
                    --namespace qa \
                    -f k3s/qa-values.yaml \
                    --set image.tag=${env.BUILD_NUMBER}
                """
            }
        }

        stage('Deploy to Staging') {
            when {
                branch 'master'
            }
            steps {
                sh """
                  helm upgrade --install app-staging helm/app \
                    --namespace staging2 \
                    -f k3s/staging2-values.yaml \
                    --set image.tag=${env.BUILD_NUMBER}
                """
            }
        }

        stage('Deploy to Prod (manuel, master uniquement)') {
            when {
                branch 'master'
            }
            steps {
                script {
                    input message: "Confirmer le déploiement en PRODUCTION ?"
                    sh """
                      helm upgrade --install app-prod helm/app \
                        --namespace prod2 \
                        -f k3s/prod2-values.yaml \
                        --set image.tag=${env.BUILD_NUMBER}
                    """
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline terminé avec statut : ${currentBuild.currentResult}"
        }
    }
}


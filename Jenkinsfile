pipeline {
    agent any

    environment {
        // Utilisateur GitHub (le même que pour GHCR)
        GITHUB_USER = 'ton_utilisateur_github'
        // Nom de ton image
        IMAGE_NAME = 'mon-app'
        // Tag automatique basé sur le numéro de build
        IMAGE_TAG = "v${env.BUILD_NUMBER}"
        // URL complète de l’image sur GHCR
        IMAGE_URI = "ghcr.io/${GITHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}"
    }

    stages {
        stage('Checkout code') {
            steps {
                echo "Clonage du dépôt GitHub..."
                checkout scm
            }
        }

        stage('Build Docker image') {
            steps {
                echo "Construction de l'image Docker..."
                sh """
                docker build -t ${IMAGE_URI} .
                """
            }
        }

        stage('Login to GitHub Container Registry') {
            steps {
                script {
                    echo "Connexion à GitHub Container Registry..."
                    // Le token GitHub doit avoir le scope `write:packages` et `read:packages`
                    withCredentials([string(credentialsId: 'github-token', variable: 'GH_TOKEN')]) {
                        sh "echo $GH_TOKEN | docker login ghcr.io -u ${GITHUB_USER} --password-stdin"
                    }
                }
            }
        }

        stage('Push Docker image') {
            steps {
                echo "Poussée de l'image vers GitHub Container Registry..."
                sh """
                docker push ${IMAGE_URI}
                docker tag ${IMAGE_URI} ghcr.io/${GITHUB_USER}/${IMAGE_NAME}:latest
                docker push ghcr.io/${GITHUB_USER}/${IMAGE_NAME}:latest
                """
            }
        }

        stage('Cleanup') {
            steps {
                echo "Nettoyage des images locales..."
                sh "docker rmi ${IMAGE_URI} ghcr.io/${GITHUB_USER}/${IMAGE_NAME}:latest || true"
            }
        }
    }

    post {
        success {
            echo "✅ Image poussée avec succès : ${IMAGE_URI}"
        }
        failure {
            echo "❌ Le pipeline a échoué."
        }
    }
}

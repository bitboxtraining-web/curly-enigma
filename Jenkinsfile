pipeline {
    agent any

    environment {
        PROJECT_NAME = "curly-enigma"

        // Slack / Teams Webhook (déjà configuré dans Jenkins Credentials)
        SLACK_WEBHOOK = credentials('slack-webhook')
        TEAMS_WEBHOOK = credentials('teams-webhook')

        // SonarQube
        SONARQUBE_SERVER = "sonarqube"
        SONARQUBE_TOKEN = credentials('sonar-token')
    }

    stages {

        stage('Checkout') {
            steps {
                echo "📥 Récupération du code source..."
                checkout scm
                echo "Branche détectée : ${env.BRANCH_NAME}"
            }
        }

        stage('Install Dependencies') {
            when {
                expression { fileExists('package.json') }
            }
            steps {
                echo "📦 Installation des dépendances..."
                sh "npm install"
            }
        }

        stage('Run Tests') {
            when {
                anyOf {
                    branch 'develop'
                    expression { env.BRANCH_NAME.startsWith('feature/') }
                    expression { env.BRANCH_NAME.startsWith('release/') }
                }
            }
            steps {
                echo "🧪 Exécution des tests..."
                sh "npm test --if-present"
                junit 'reports/junit/*.xml'
            }
        }

        stage('Static Code Analysis') {
            when {
                anyOf {
                    branch 'develop'
                    branch 'main'
                    expression { env.BRANCH_NAME.startsWith('release/') }
                }
            }
            steps {
                echo "🔍 Contrôle qualité avec SonarQube..."
                withSonarQubeEnv("${SONARQUBE_SERVER}") {
                    sh "npm run sonar || true"
                }
            }
        }

        stage('Build Artifact / Image') {
            when {
                anyOf {
                    branch 'main'
                    branch 'develop'
                    expression { env.BRANCH_NAME.startsWith('release/') }
                }
            }
            steps {
                echo "🏗 Construction de l’artifact ou image Docker..."
                sh "docker build -t ${PROJECT_NAME}:${env.BUILD_NUMBER} ."
            }
        }

        stage('Deploy to Staging (Release branches)') {
            when {
                expression { env.BRANCH_NAME.startsWith('release/') }
            }
            steps {
                echo "🚀 Déploiement en environnement staging..."
                sh "echo Deploy STAGING"
            }
        }

        stage('Deploy to Production (Main)') {
            when {
                branch 'main'
            }
            steps {
                echo "🔥 Déploiement en production..."
                sh "echo Deploy PRODUCTION"
            }
        }

    }

    post {

        success {
            echo "✅ Pipeline réussi pour la branche : ${env.BRANCH_NAME}"

            // Slack
            sh """
            curl -X POST -H 'Content-type: application/json' \
            --data '{"text":"✅ Build *SUCCESS* for branch *${env.BRANCH_NAME}* : ${env.BUILD_URL}"}' \
            $SLACK_WEBHOOK
            """

            // Teams
            sh """
            curl -H 'Content-Type: application/json' \
            -d '{"text": "✅ Success: Branch ${env.BRANCH_NAME}"}' \
            $TEAMS_WEBHOOK
            """

            // Email
            mail to: 'team@company.com',
                 subject: "✅ SUCCESS - ${PROJECT_NAME} - ${env.BRANCH_NAME}",
                 body: "Le pipeline a réussi.\nBuild: ${env.BUILD_URL}"
        }

        failure {

            echo "❌ Pipeline échoué pour la branche : ${env.BRANCH_NAME}"

            // Slack
            sh """
            curl -X POST -H 'Content-type: application/json' \
            --data '{"text":"❌ Build *FAILED* for branch *${env.BRANCH_NAME}* : ${env.BUILD_URL}"}' \
            $SLACK_WEBHOOK
            """

            // Teams
            sh """
            curl -H 'Content-Type: application/json' \
            -d '{"text": "❌ ECHEC: Branch ${env.BRANCH_NAME}"}' \
            $TEAMS_WEBHOOK
            """

            // Email
            mail to: 'team@company.com',
                 subject: "❌ ECHEC - ${PROJECT_NAME} - ${env.BRANCH_NAME}",
                 body: "Le pipeline a échoué.\nBuild: ${env.BUILD_URL}"
        }
    }
}

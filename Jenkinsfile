// Jenkinsfile
pipeline {
    agent any

    // Déclenche sur push dans 'main' (ou via webhook GitHub)
    triggers {
        githubPush() // Nécessite le plugin "GitHub Branch Source"
    }

    options {
        timeout(time: 20, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '10'))
        skipDefaultCheckout(true) // On gère le checkout manuellement
    }

    environment {
        SLACK_CHANNEL = '#jenkins'
        SLACK_COLOR_DANGER = '#D50200'
        SLACK_COLOR_WARNING = '#D5A100'
        SLACK_COLOR_GOOD = '#00B309'
        GIT_REPO_URL = 'https://github.com/tonorg/tonrepo.git'
        GIT_CREDENTIALS_ID = 'git-credentials-id'
    }

    stages {
        stage('Checkout') {
            when {
                expression { env.BRANCH_NAME == 'main' }
            }
            steps {
                slackSend(
                    channel: env.SLACK_CHANNEL,
                    color: env.SLACK_COLOR_WARNING,
                    message: "*Début du stage Checkout* sur branche `${env.BRANCH_NAME}`"
                )

                checkout scmGit(
                    branches: [[name: 'main']],
                    userRemoteConfigs: [[
                        credentialsId: env.GIT_CREDENTIALS_ID,
                        url: env.GIT_REPO_URL
                    ]]
                )
            }
            post {
                success {
                    slackSend channel: env.SLACK_CHANNEL, color: env.SLACK_COLOR_GOOD,
                              message: "*Checkout* terminé avec succès !"
                }
                failure {
                    slackSend channel: env.SLACK_CHANNEL, color: env.SLACK_COLOR_DANGER,
                              message: "*Checkout* a échoué ! Vérifie les credentials Git."
                }
            }
        }

        stage('Build') {
            when {
                expression { env.BRANCH_NAME == 'main' }
            }
            steps {
                slackSend channel: env.SLACK_CHANNEL, color: env.SLACK_COLOR_WARNING,
                          message: "*Début du stage Build*"

                script {
                    if (sh(script: 'command -v mvn', returnStatus: true) != 0) {
                        error "Maven n'est pas installé sur cet agent !"
                    }
                    sh 'mvn clean package -DskipTests'
                }
            }
            post {
                success {
                    slackSend channel: env.SLACK_CHANNEL, color: env.SLACK_COLOR_GOOD,
                              message: "*Build* terminé ! Artefact généré."
                }
                failure {
                    slackSend channel: env.SLACK_CHANNEL, color: env.SLACK_COLOR_DANGER,
                              message: "*Build* a échoué !"
                }
            }
        }

        stage('Deploy') {
            when {
                expression { env.BRANCH_NAME == 'main' }
            }
            steps {
                slackSend channel: env.SLACK_CHANNEL, color: env.SLACK_COLOR_WARNING,
                          message: "*Début du stage Deploy* vers prod"

                script {
                    if (sh(script: 'command -v docker', returnStatus: true) != 0) {
                        error "Docker n'est pas installé !"
                    }
                    sh '''
                        docker build -t monapp:latest .
                        docker push registry/monapp:latest
                    '''
                }
            }
            post {
                success {
                    slackSend channel: env.SLACK_CHANNEL, color: env.SLACK_COLOR_GOOD,
                              message: "*Deploy* terminé ! App en prod."
                }
                failure {
                    slackSend channel: env.SLACK_CHANNEL, color: env.SLACK_COLOR_DANGER,
                              message: "*Deploy* a échoué !"
                }
            }
        }
    }

    post {
        always {
            script {
                def color = currentBuild.result == 'SUCCESS' ? env.SLACK_COLOR_GOOD : env.SLACK_COLOR_DANGER
                slackSend(
                    channel: env.SLACK_CHANNEL,
                    color: color,
                    message: "*Pipeline terminé* `${env.JOB_NAME}` #${env.BUILD_NUMBER}\n" +
                             "Branche: `${env.BRANCH_NAME}`\n" +
                             "Résultat: *${currentBuild.result ?: 'UNKNOWN'}*\n" +
                             "Durée: ${currentBuild.durationString}"
                )
            }
        }
    }
}
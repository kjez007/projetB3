// Jenkinsfile
pipeline {
    agent any

    triggers {
        githubPush() // Déclenche sur push GitHub
    }

    options {
        timeout(time: 20, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '10'))
        skipDefaultCheckout(true)
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

        // STAGE CLONE (comme tu l'as demandé)
        stage('Clone') {
            when {
                expression { env.BRANCH_NAME == 'main' }
            }
            steps {
                slackSend(
                    channel: env.SLACK_CHANNEL,
                    color: env.SLACK_COLOR_WARNING,
                    message: "*Début du stage Clone* du dépôt `${env.GIT_REPO_URL}` sur branche `${env.BRANCH_NAME}`"
                )

                // CLONE RÉEL
                checkout scmGit(
                    branches: [[name: 'main']],
                    userRemoteConfigs: [[
                        credentialsId: env.GIT_CREDENTIALS_ID,
                        url: env.GIT_REPO_URL
                    ]]
                )

                // Preuve dans les logs
                sh 'echo "Contenu du workspace après clone :"'
                sh 'ls -la'
                sh 'git log --oneline -5'
            }
            post {
                success {
                    slackSend(
                        channel: env.SLACK_CHANNEL,
                        color: env.SLACK_COLOR_GOOD,
                        message: "*Clone* terminé avec succès ! Code récupéré."
                    )
                }
                failure {
                    slackSend(
                        channel: env.SLACK_CHANNEL,
                        color: env.SLACK_COLOR_DANGER,
                        message: "*Clone* a échoué ! Vérifie l’URL ou les credentials."
                    )
                }
            }
        }

        stage('Build') {
            when { expression { env.BRANCH_NAME == 'main' } }
            steps {
                slackSend channel: env.SLACK_CHANNEL, color: env.SLACK_COLOR_WARNING,
                          message: "*Début du stage Build*"

                script {
                    if (sh(script: 'command -v mvn', returnStatus: true) != 0) {
                        error "Maven non installé !"
                    }
                    sh 'mvn clean package -DskipTests'
                }
            }
            post {
                success { slackSend channel: env.SLACK_CHANNEL, color: env.SLACK_COLOR_GOOD, message: "*Build* OK" }
                failure { slackSend channel: env.SLACK_CHANNEL, color: env.SLACK_COLOR_DANGER, message: "*Build* échoué" }
            }
        }

        stage('Deploy') {
            when { expression { env.BRANCH_NAME == 'main' } }
            steps {
                slackSend channel: env.SLACK_CHANNEL, color: env.SLACK_COLOR_WARNING,
                          message: "*Début du stage Deploy*"

                script {
                    if (sh(script: 'command -v docker', returnStatus: true) != 0) {
                        error "Docker non installé !"
                    }
                    sh '''
                        docker build -t monapp:latest .
                        docker push registry/monapp:latest
                    '''
                }
            }
            post {
                success { slackSend channel: env.SLACK_CHANNEL, color: env.SLACK_COLOR_GOOD, message: "*Deploy* OK" }
                failure { slackSend channel: env.SLACK_CHANNEL, color: env.SLACK_COLOR_DANGER, message: "*Deploy* échoué" }
            }
        }
    }

    post {
        always {
            def color = currentBuild.result == 'SUCCESS' ? env.SLACK_COLOR_GOOD : env.SLACK_COLOR_DANGER
            slackSend(
                channel: env.SLACK_CHANNEL,
                color: color,
                message: """
*Pipeline terminé* `${env.JOB_NAME}` #${env.BUILD_NUMBER}
Branche: `${env.BRANCH_NAME}`
Résultat: *${currentBuild.result ?: 'UNKNOWN'}*
Durée: ${currentBuild.durationString}
                """.stripIndent()
            )
        }
    }
}
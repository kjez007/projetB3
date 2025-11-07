// Jenkinsfile
pipeline {
    agent any

    // Déclenchement uniquement sur la branche "main" (ou celle que tu veux)
    triggers {
        pollSCM('') // ou githubPush() si webhook GitHub
    }

    options {
        timeout(time: 20, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    environment {
        SLACK_CHANNEL = '#jenkins-ci'
        SLACK_COLOR_DANGER = '#D50200'
        SLACK_COLOR_WARNING = '#D5A100'
        SLACK_COLOR_GOOD = '#00B309'
    }

    stages {
        stage('Clone') {
            steps {
                slackSend(
                    channel: env.SLACK_CHANNEL,
                    color: env.SLACK_COLOR_WARNING,
                    message: "*Début du stage Clone* sur branche `${env.BRANCH_NAME}`"
                )
                git branch: 'main',
                    credentialsId: 'git-credentials-id',
                    url: 'https://github.com/tonorg/tonrepo.git'
            }
            post {
                success {
                    slackSend(
                        channel: env.SLACK_CHANNEL,
                        color: env.SLACK_COLOR_GOOD,
                        message: "*Clone* terminé avec succès !"
                    )
                }
                failure {
                    slackSend(
                        channel: env.SLACK_CHANNEL,
                        color: env.SLACK_COLOR_DANGER,
                        message: "*Clone* a échoué !"
                    )
                }
            }
        }

        stage('Build') {
            steps {
                slackSend(
                    channel: env.SLACK_CHANNEL,
                    color: env.SLACK_COLOR_WARNING,
                    message: "*Début du stage Build*"
                )
                // Exemple Maven
                sh 'mvn clean package -DskipTests'
            }
            post {
                success {
                    slackSend(
                        channel: env.SLACK_CHANNEL,
                        color: env.SLACK_COLOR_GOOD,
                        message: "*Build* terminé avec succès ! Artefact généré."
                    )
                }
                failure {
                    slackSend(
                        channel: env.SLACK_COLOR_DANGER,
                        message: "*Build* a échoué !"
                    )
                }
            }
        }

        stage('Deploy') {
            when {
                branch 'main'   // ne s’exécute que sur main
            }
            steps {
                slackSend(
                    channel: env.SLACK_CHANNEL,
                    color: env.SLACK_COLOR_WARNING,
                    message: "*Début du stage Deploy* vers prod"
                )
                // Exemple : déploiement Docker
                sh '''
                    docker build -t monapp:latest .
                    docker push registry/monapp:latest
                '''
            }
            post {
                success {
                    slackSend(
                        channel: env.SLACK_CHANNEL,
                        color: env.SLACK_COLOR_GOOD,
                        message: "*Deploy* terminé ! Application en prod."
                    )
                }
                failure {
                    slackSend(
                        channel: env.SLACK_COLOR_DANGER,
                        message: "*Deploy* a échoué !"
                    )
                }
            }
        }
    }

    post {
        always {
            slackSend(
                channel: env.SLACK_CHANNEL,
                color: (currentBuild.result == 'SUCCESS' ? env.SLACK_COLOR_GOOD : env.SLACK_COLOR_DANGER),
                message: "*Pipeline terminé* `${env.JOB_NAME}` #${env.BUILD_NUMBER} — Résultat : *${currentBuild.result}*"
            )
        }
    }
}
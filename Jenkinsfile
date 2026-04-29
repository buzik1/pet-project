pipeline {
    agent any

    triggers {
        githubPush()
    }

    environment {
        ANSIBLE_HOST_KEY_CHECKING = 'false'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                echo 'Code checked out from Git.'
            }
        }

        stage('Run Ansible Playbook') {
            steps {
                echo 'Deploying application using Ansible with Vault...'
                dir('ansible') {
                    withCredentials([string(credentialsId: 'ansible-vault-password', variable: 'VAULT_PASS')]) {
                        sh '''
                            # Создаём временный файл с паролем Vault
                            echo "$VAULT_PASS" > ../.vault_pass
                            # Запускаем плейбук с передачей пароля
                            ansible-playbook -i inventory/production.yml playbooks/deploy.yml --vault-password-file ../.vault_pass
                            # Удаляем временный файл с паролем
                            rm -f ../.vault_pass
                        '''
                    }
                }
            }
        }
    }

    post {
        success {
            slackSend(
                tokenCredentialId: 'builds_bot_token',
                channel: '#builds',
                color: 'good',
                message: "✅ Сборка *${env.JOB_NAME}* #${env.BUILD_NUMBER} прошла успешно.\n${env.BUILD_URL}"
            )
            echo 'Deployment successful!'
        }
        failure {
            slackSend(
                tokenCredentialId: 'builds_bot_token',
                channel: '#builds',
                color: 'danger',
                message: "❌ Сборка *${env.JOB_NAME}* #${env.BUILD_NUMBER} завершилась неудачей.\n${env.BUILD_URL}"
            )
            echo 'Deployment failed. Check the logs.'
        }
    }
}
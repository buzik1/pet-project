pipeline {
    agent any

    triggers {
        githubPush()
    }

    environment {
        ANSIBLE_HOST_KEY_CHECKING = 'false'
        ANSIBLE_SSH_KEY = '/var/lib/jenkins/.ssh/ansible_key'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                echo 'Code checked out from Git.'
            }
        }

        // ✅ Новая стадия: получаем сообщение последнего коммита
        stage('Get Commit Info') {
            steps {
                script {
                    env.GIT_COMMIT_MSG = sh(
                        script: "git log -1 --pretty=format:%s",
                        returnStdout: true
                    ).trim()
                    .replace('"', '\\"')
                    .replace('`', '\\`')
                }
            }
        }

        stage('Validate YAML configs') {
            steps {
                echo 'Checking YAML syntax...'
                sh '''
                    find . -type f \\( -name "*.yml" -o -name "*.yaml" \\) ! -name "vault.yml" -print0 | xargs -0 yamllint -d relaxed
                    if [ $? -ne 0 ]; then
                        echo "❌ YAML syntax errors found! Fix them before deploying."
                        exit 1
                    fi
                    echo "✅ All YAML files look good."
                '''
            }
        }

        stage('Run Ansible Playbook') {
            steps {
                echo 'Deploying application using Ansible with Vault...'
                dir('ansible') {
                    withCredentials([string(credentialsId: 'ansible-vault-password', variable: 'VAULT_PASS')]) {
                        sh '''
                            echo "$VAULT_PASS" > ../.vault_pass
                            ansible-playbook -i inventory/production.yml playbooks/deploy.yml --vault-password-file ../.vault_pass
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
                message: "✅ Сборка *${env.JOB_NAME}* #${env.BUILD_NUMBER} прошла успешно.\nCommit: `${env.GIT_COMMIT_MSG}`\n${env.BUILD_URL}"
            )
            echo 'Deployment successful!'
        }
        failure {
            slackSend(
                tokenCredentialId: 'builds_bot_token',
                channel: '#builds',
                color: 'danger',
                message: "❌ Сборка *${env.JOB_NAME}* #${env.BUILD_NUMBER} завершилась неудачей.\nCommit: `${env.GIT_COMMIT_MSG}`\n${env.BUILD_URL}"
            )
            echo 'Deployment failed. Check the logs.'
        }
    }
}
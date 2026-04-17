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
                echo 'Deploying application using Ansible...'
                dir('ansible') {
                    sh '''
                    ansible-playbook -i inventory/production.yml playbooks/deploy.yml
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment successful!'
        }
        failure {
            echo 'Deployment failed. Check the logs.'
        }
    }
}
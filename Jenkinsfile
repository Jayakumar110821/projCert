pipeline {
    agent any

    environment {
        SLAVE_HOST = '172.31.1.32'
        SLAVE_USER = 'edureka'
        DOCKER_IMAGE_NAME = 'php-app' // You can choose a different name
        CONTAINER_NAME = 'php-app-container'
    }

    stages {
      stage('Install Puppet Agent') {
    steps {
        sshagent(credentials: ['slave-ssh-key']) {
            sh 'env'
            sh "ssh -o StrictHostKeyChecking=no ${SLAVE_USER}@${SLAVE_HOST} 'sudo apt update && sudo apt install -y puppet-agent'"
        }
    }
}

        stage('Install Docker via Ansible') {
            steps {
                script {
                    ansiblePlaybook credentialsId: 'slave-ssh-key', // Use the credential ID
                                    inventory: "${SLAVE_HOST},",
                                    playbook: 'ansible/install_docker.yml',
                                    sudo: true,
                                    extraVars: [ansible_user: "${SLAVE_USER}"]
                }
            }
        }

        stage('Build and Deploy Docker Container') {
            steps {
                git url: 'https://github.com/Jayakumar110821/projCert.git',
                    branch: 'master',
                    credentialsId: '' // Only needed if your repo is private

                script {
                    sh "docker build -t ${DOCKER_IMAGE_NAME} ."
                    sshagent(credentials: 'slave-ssh-key') {
                        sh "ssh -o StrictHostKeyChecking=no ${SLAVE_USER}@${SLAVE_HOST} 'docker stop ${CONTAINER_NAME} && docker rm ${CONTAINER_NAME} && docker run -d --name ${CONTAINER_NAME} -p 80:80 ${DOCKER_IMAGE_NAME}'"
                    }
                }
            }
            post {
                failure {
                    sshagent(credentials: 'slave-ssh-key') {
                        sh "ssh -o StrictHostKeyChecking=no ${SLAVE_USER}@${SLAVE_HOST} 'docker stop ${CONTAINER_NAME} && docker rm ${CONTAINER_NAME}'"
                    }
                }
            }
        }
    }
}

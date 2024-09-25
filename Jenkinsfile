pipeline {
    agent any

    environment {
        NODE_ENV = 'production'
        DEPLOY_USER = 'ubuntu'
        DEPLOY_HOST = '18.220.0.151'
        DEPLOY_PATH = '/home/ubuntu/new-test'
    }

    stages {
        stage('Checkout Code') {
            steps {
                script {
                    git url: 'https://github.com/pankaj200117/new-test.git', credentialsId: 'github_token'
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                script {
                    sh 'npm install'
                }
            }
        }

        stage('Build the App') {
            steps {
                script {
                    sh 'npm run build'
                }
            }
        }

        stage('Debug SSH Connection') {
            steps {
                script {
                    withCredentials([sshUserPrivateKey(credentialsId: 'ssh_key', keyFileVariable: 'SSH_KEY')]) {
                        sh """
                            ssh -o StrictHostKeyChecking=no -i \$SSH_KEY ${DEPLOY_USER}@${DEPLOY_HOST} 'whoami; ls -la; pwd'
                        """
                    }
                }
            }
        }

        stage('Deploy to Server') {
            steps {
                script {
                    withCredentials([sshUserPrivateKey(credentialsId: 'ssh_key', keyFileVariable: 'SSH_KEY')]) {
                        // Set permissions for the private key
                        sh 'chmod 600 $SSH_KEY'

                        // Create directory on the server
                        sh """
                            ssh -o StrictHostKeyChecking=no -i \$SSH_KEY ${DEPLOY_USER}@${DEPLOY_HOST} 'mkdir -p ${DEPLOY_PATH}'
                        """

                        // Copy files to the server
                        sh """
                            rsync -avz --delete ./build/ ${DEPLOY_USER}@${DEPLOY_HOST}:${DEPLOY_PATH}/
                        """

                        // Restart the application on the server
                        sh """
                            ssh -o StrictHostKeyChecking=no -i \$SSH_KEY ${DEPLOY_USER}@${DEPLOY_HOST} 'cd ${DEPLOY_PATH} && npm install && pm2 restart all'
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment successful!'
        }
        failure {
            echo 'Deployment failed.'
        }
    }
}


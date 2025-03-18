pipeline {
    agent { label 'jenkins-runner1' }

    environment {
        DEPLOY_SERVER = "10.247.109.112"  
        DEPLOY_USER = "ubuntu" 
        DEPLOY_PATH = "/var/www/sample-node-project" 
        CREDENTIAL_ID ="app-server-ssh-key" 
    }

    stages {
        stage('Install Dependencies') {
            steps {
                nodejs('nodejs18') {
                 echo "Installing dependencies"
                 sh 'npm install'
                 
                }
            }
        }

        stage('Build Application') {
                steps {
                    nodejs('nodejs18') {
                        echo "Building Code"
                        #sh 'npm run build'
                    }
                }
        }

        stage('Deploy to Server') {
            steps {
                script {
                    echo "Deploying application to EC2..."
                    sshagent([CREDENTIAL_ID]) {
                        sh """
                            ssh -o StrictHostKeyChecking=no ${DEPLOY_USER}@${DEPLOY_SERVER} '
                            mkdir -p ${DEPLOY_PATH} &&
                            rm -rf ${DEPLOY_PATH}/* &&
                            exit'
                        """
                        sh "scp -r * ${DEPLOY_USER}@${DEPLOY_SERVER}:${DEPLOY_PATH}/"
                    }
                }
            }
        }

        stage('Install & Run with PM2') {
            steps {
                script {
                    echo "Installing PM2 & starting the application..."
                    sshagent([CREDENTIAL_ID]) {
                        sh """
                            ssh -o StrictHostKeyChecking=no ${DEPLOY_USER}@${DEPLOY_SERVER} '
                            if [ -d "${DEPLOY_PATH}" ] then
                                cd "${DEPLOY_PATH}" &&
                                ls  -l
                            else
                                mkdir -p "${DEPLOY_PATH}"
                                pwd
                            fi
                            
                            exit'
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo '✅ Deployment completed successfully!'
        }
        failure {
            echo '❌ Deployment failed! Please check the logs.'
        }
    }
}

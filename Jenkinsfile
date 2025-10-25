pipeline {
    agent any
    
    environment {
        DEPLOY_SERVER = '143.1.1.65'
        DEPLOY_PORT = '8111'
        APP_DIR = '/opt/project-management'
    }
    
    stages {
        stage('Deploy to FastAPI Server') {
            steps {
                script {
                    echo "Deploying to ${DEPLOY_SERVER}"
                    
                    withCredentials([sshUserPrivateKey(
                        credentialsId: 'deploy-to-65',
                        keyFileVariable: 'SSH_KEY',
                        usernameVariable: 'SSH_USER'
                    )]) {
                        sh """
                            # Test connection first
                            ssh -o StrictHostKeyChecking=no -i \${SSH_KEY} \${SSH_USER}@${DEPLOY_SERVER} 'echo "✅ SSH connection successful"'
                            
                            # Create directory on target server
                            ssh -o StrictHostKeyChecking=no -i \${SSH_KEY} \${SSH_USER}@${DEPLOY_SERVER} "
                                mkdir -p ${APP_DIR}
                                chmod 755 ${APP_DIR}
                                echo '✅ Directory created successfully'
                            "
                            
                            # Copy files using rsync
                            rsync -avz -e "ssh -o StrictHostKeyChecking=no -i \${SSH_KEY}" \\
                                --exclude='__pycache__' \\
                                --exclude='venv' \\
                                --exclude='.git' \\
                                --exclude='*.log' \\
                                --exclude='*.pid' \\
                                --exclude='node_modules' \\
                                ./ \${SSH_USER}@${DEPLOY_SERVER}:${APP_DIR}/
                            
                            echo '✅ Files copied successfully'
                            
                            # Deploy application
                            ssh -o StrictHostKeyChecking=no -i \${SSH_KEY} \${SSH_USER}@${DEPLOY_SERVER} "
                                cd ${APP_DIR}
                                echo 'Stopping existing application...'
                                pkill -f 'uvicorn' || true
                                sleep 3
                                echo 'Installing Python dependencies...'
                                pip3 install -r requirements.txt
                                echo 'Starting FastAPI application...'
                                nohup python3 -m uvicorn main:app --host 0.0.0.0 --port ${DEPLOY_PORT} > app.log 2>&1 &
                                echo '✅ Application deployment completed!'
                            "
                        """
                    }
                }
            }
        }
        
        stage('Verify Deployment') {
            steps {
                script {
                    sleep 8  # Give more time for app to start
                    sh """
                        if curl -s -f http://${DEPLOY_SERVER}:${DEPLOY_PORT}/docs; then
                            echo '✅ Deployment verified successfully!'
                        else
                            echo '❌ Deployment verification failed'
                            # Check if app is running
                            ssh -o StrictHostKeyChecking=no -i \${SSH_KEY} \${SSH_USER}@${DEPLOY_SERVER} "ps aux | grep uvicorn"
                            exit 1
                        fi
                    """
                }
            }
        }
    }
    
    post {
        always {
            echo "Deployment process completed for ${DEPLOY_SERVER}"
        }
        success {
            echo "🎉 Successfully deployed to ${DEPLOY_SERVER}:${DEPLOY_PORT}"
        }
        failure {
            echo "💥 Deployment failed - check logs above"
        }
    }
}

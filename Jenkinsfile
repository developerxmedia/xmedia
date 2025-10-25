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
                    
                    sshagent(['deploy-to-65']) {
                        // Create directory on target server
                        sh """
                            ssh -o StrictHostKeyChecking=no root@${DEPLOY_SERVER} "
                                mkdir -p ${APP_DIR}
                                chmod 755 ${APP_DIR}
                            "
                        """
                        
                        // Copy files using rsync (more efficient)
                        sh """
                            rsync -avz -e "ssh -o StrictHostKeyChecking=no" \
                            --exclude='__pycache__' \
                            --exclude='venv' \
                            --exclude='.git' \
                            --exclude='*.log' \
                            --exclude='*.pid' \
                            ./ root@${DEPLOY_SERVER}:${APP_DIR}/
                        """
                        
                        // Deploy application
                        sh """
                            ssh -o StrictHostKeyChecking=no root@${DEPLOY_SERVER} "
                                cd ${APP_DIR}
                                echo 'Stopping existing application...'
                                pkill -f 'uvicorn' || true
                                sleep 3
                                echo 'Installing Python dependencies...'
                                pip3 install -r requirements.txt
                                echo 'Starting FastAPI application...'
                                nohup python3 -m uvicorn main:app --host 0.0.0.0 --port ${DEPLOY_PORT} > app.log 2>&1 &
                                echo 'Application deployment completed!'
                            "
                        """
                    }
                }
            }
        }
        
        stage('Verify Deployment') {
            steps {
                script {
                    sleep 5
                    sh """
                        curl -s -f http://${DEPLOY_SERVER}:${DEPLOY_PORT}/docs > /dev/null && echo '✅ Deployment successful!'
                    """
                }
            }
        }
    }
}

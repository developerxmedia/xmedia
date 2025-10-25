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
                    echo "Deploying to ${DEPLOY_SERVER}:${DEPLOY_PORT}"
                    
                    // Copy files to target server
                    sh """
                        scp -o StrictHostKeyChecking=no -r ./* ${DEPLOY_SERVER}:${APP_DIR}/
                    """
                    
                    // Run deployment commands on target server
                    sshagent(['your-ssh-credentials-id']) {
                        sh """
                            ssh -o StrictHostKeyChecking=no ${DEPLOY_SERVER} '
                                cd ${APP_DIR}
                                echo "Installing dependencies..."
                                pip3 install -r requirements.txt
                                echo "Starting application..."
                                nohup python3 -m uvicorn main:app --host 0.0.0.0 --port ${DEPLOY_PORT} > app.log 2>&1 &
                                echo "Application deployed successfully"
                            '
                        """
                    }
                }
            }
        }
    }
    
    post {
        always {
            echo "Deployment process completed"
        }
        success {
            echo "✅ Application successfully deployed to ${DEPLOY_SERVER}:${DEPLOY_PORT}"
        }
        failure {
            echo "❌ Deployment failed - check logs above"
        }
    }
}

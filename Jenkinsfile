pipeline {
    agent any
    
    environment {
        // Target server where FastAPI will run
        DEPLOY_SERVER = '143.1.1.65'
        DEPLOY_PORT = '8111'
        APP_DIR = '/opt/fastapi-app'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Test') {
            steps {
                sh '''
                    echo "Running tests..."
                    python -m pytest tests/ -v
                '''
            }
        }
        
        stage('Deploy to Server') {
            steps {
                script {
                    // Transfer files to target server
                    sh """
                        scp -o StrictHostKeyChecking=no -r ./* ${DEPLOY_SERVER}:${APP_DIR}/
                    """
                    
                    // Execute deployment commands on target server
                    sshagent(['your-ssh-credentials-id']) {
                        sh """
                            ssh -o StrictHostKeyChecking=no ${DEPLOY_SERVER} '
                                cd ${APP_DIR}
                                python -m pip install -r requirements.txt
                                sudo systemctl daemon-reload
                                sudo systemctl restart fastapi-app
                                sudo systemctl status fastapi-app
                            '
                        """
                    }
                }
            }
        }
        
        stage('Health Check') {
            steps {
                script {
                    // Verify the application is running
                    sh """
                        curl -f http://${DEPLOY_SERVER}:${DEPLOY_PORT}/health || exit 1
                    """
                    echo "Deployment completed successfully!"
                }
            }
        }
    }
    
    post {
        always {
            echo "Pipeline execution completed"
        }
        success {
            echo "Application deployed successfully to ${DEPLOY_SERVER}"
        }
        failure {
            echo "Deployment failed - check the logs"
        }
    }
}

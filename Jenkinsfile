pipeline {
    agent any

    environment {
        VENV_DIR = 'venv'
        APP_PORT = '8111'
        APP_HOST = '0.0.0.0'
        APP_ENTRY_POINT = 'main:app'
        JENKINS_IP = '143.1.1.128'
    }

    options {
        timeout(time: 30, unit: 'MINUTES')
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    echo "🐍 Setting up environment..."
                    python3 -m venv ${VENV_DIR}
                    . ${VENV_DIR}/bin/activate
                    pip install --upgrade pip
                    pip install -r requirements.txt || pip install fastapi uvicorn
                '''
            }
        }

        stage('Deploy and Test Network') {
            steps {
                sh '''
                    . ${VENV_DIR}/bin/activate
                    echo "🚀 Starting: ${APP_ENTRY_POINT}"
                    
                    # Stop any previous instance
                    if [ -f "app.pid" ]; then
                        echo "🛑 Stopping previous instance..."
                        kill $(cat app.pid) 2>/dev/null || true
                        rm -f app.pid
                    fi
                    
                    # Start the application
                    nohup python -m uvicorn ${APP_ENTRY_POINT} --host ${APP_HOST} --port ${APP_PORT} > app.log 2>&1 &
                    echo $! > app.pid
                    sleep 5
                    
                    echo "📝 Application logs:"
                    tail -n 10 app.log
                    
                    echo "🔍 Network Diagnostics:"
                    echo "1. Checking if process is running:"
                    ps -p $(cat app.pid) && echo "✅ Process running" || echo "❌ Process not running"
                    
                    echo "2. Checking port binding:"
                    netstat -tulpn | grep :${APP_PORT} || ss -tulpn | grep :${APP_PORT} || echo "⚠️ Cannot check port binding"
                    
                    echo "3. Testing from inside container:"
                    curl -f http://localhost:${APP_PORT}/ && echo "✅ Local access works" || echo "❌ Local access failed"
                    
                    echo "4. Testing from container IP:"
                    curl -f http://127.0.0.1:${APP_PORT}/ && echo "✅ 127.0.0.1 access works" || echo "❌ 127.0.0.1 access failed"
                    
                    echo "5. Testing from 0.0.0.0:"
                    curl -f http://0.0.0.0:${APP_PORT}/ && echo "✅ 0.0.0.0 access works" || echo "❌ 0.0.0.0 access failed"
                    
                    echo "6. Container IP address:"
                    hostname -I || ip addr show || echo "⚠️ Cannot get IP"
                    
                    echo "🌐 If local tests work but external access fails, check:"
                    echo "   - Docker port mapping"
                    echo "   - Firewall settings"
                    echo "   - Network routing"
                '''
            }
        }
    }

    post {
        always {
            echo "🏁 Pipeline completed"
            echo "💡 Application running inside container - check network configuration"
        }
    }
}

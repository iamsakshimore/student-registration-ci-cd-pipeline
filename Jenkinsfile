pipeline {
    agent any

    environment {
        VENV = 'venv'
        FLASK_LOG = 'flask.log'
        SOCAT_LOG = 'socat.log'
        PORT = '5000'
        EXPOSE_PORT = '5050'
    }

    stages {

        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/iamsakshimore/stud-reg-flask-app.git'
            }
        }

        stage('Clean Workspace') {
            steps {
                sh '''
                echo "Cleaning workspace..."
                rm -rf $VENV flask.log socat.log flask_pid.txt socat_pid.txt || true
                '''
            }
        }

        stage('Install System Dependencies') {
            steps {
                sh '''
                echo "Installing system dependencies..."

                # Install Python venv + pip
                sudo apt-get update
                sudo apt-get install -y python3.12-venv python3-pip

                # Install socat only if not present
                if ! command -v socat >/dev/null 2>&1; then
                    sudo apt-get install -y socat
                fi
                '''
            }
        }

        stage('Setup Python Environment') {
            steps {
                sh '''
                echo "Setting up Python virtual environment..."

                python3 --version
                python3 -m venv $VENV

                # Use direct path instead of activate
                $VENV/bin/pip install --upgrade pip
                $VENV/bin/pip install -r requirements.txt
                '''
            }
        }

        stage('Run Flask App') {
            steps {
                sh '''
                echo "Starting Flask application..."

                pkill -f app.py || true

                nohup $VENV/bin/python3 app.py > $FLASK_LOG 2>&1 &
                echo $! > flask_pid.txt

                sleep 10

                echo "===== Flask Logs ====="
                cat $FLASK_LOG || true

                # Check if Flask started
                ps -p $(cat flask_pid.txt) > /dev/null || {
                    echo "Flask app failed to start!"
                    exit 1
                }
                '''
            }
        }

        stage('Expose App using socat') {
            steps {
                sh '''
                echo "Starting socat..."

                pkill socat || true

                nohup socat TCP-LISTEN:$EXPOSE_PORT,fork TCP:localhost:$PORT > $SOCAT_LOG 2>&1 &
                echo $! > socat_pid.txt

                sleep 5

                echo "===== Socat Logs ====="
                cat $SOCAT_LOG || true
                '''
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                echo "Checking application health..."

                for i in {1..10}; do
                    if curl -s http://localhost:$EXPOSE_PORT > /dev/null; then
                        echo "Application is UP!"
                        exit 0
                    fi
                    echo "Retrying..."
                    sleep 3
                done

                echo "Application failed health check!"
                exit 1
                '''
            }
        }
    }

    post {
        always {
            sh '''
            echo "Cleaning up processes..."
            pkill -f app.py || true
            pkill socat || true
            '''
        }

        success {
            echo "Deployment Successful!"
        }

        failure {
            echo "Deployment Failed! Check logs above."
        }
    }
}
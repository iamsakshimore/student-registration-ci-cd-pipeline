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

        stage('Setup Python Environment') {
            steps {
                sh '''
                echo "Setting up Python environment..."

                python3 --version
                python3 -m venv $VENV

                $VENV/bin/pip install --upgrade pip
                $VENV/bin/pip install -r requirements.txt
                '''
            }
        }

        stage('Run Flask App') {
            steps {
                sh '''
                echo "Starting Flask app..."

                pkill -f app.py || true

                nohup $VENV/bin/python3 app.py > $FLASK_LOG 2>&1 &
                echo $! > flask_pid.txt

                sleep 10

                echo "===== Flask Logs ====="
                cat $FLASK_LOG || true

                ps -p $(cat flask_pid.txt) > /dev/null || {
                    echo "Flask app failed!"
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
                echo "Checking application..."

                for i in {1..10}; do
                    if curl -s http://localhost:$EXPOSE_PORT > /dev/null; then
                        echo "App is UP!"
                        exit 0
                    fi
                    echo "Retrying..."
                    sleep 3
                done

                echo "Health check failed!"
                exit 1
                '''
            }
        }
    }

    post {
        always {
            sh '''
            echo "Cleaning up..."
            pkill -f app.py || true
            pkill socat || true
            '''
        }

        success {
            echo "Deployment Successful!"
        }

        failure {
            echo "Deployment Failed!"
        }
    }
}
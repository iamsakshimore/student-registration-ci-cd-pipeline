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
                rm -rf $VENV flask.log socat.log flask_pid.txt socat_pid.txt || true
                '''
            }
        }

        stage('Setup Python Environment') {
            steps {
                sh '''
                python3 --version
                python3 -m venv $VENV

                . $VENV/bin/activate
                pip install --upgrade pip
                pip install -r requirements.txt
                '''
            }
        }

        stage('Install System Dependencies') {
            steps {
                sh '''
                which socat || sudo apt-get update && sudo apt-get install -y socat
                '''
            }
        }

        stage('Run Flask App') {
            steps {
                sh '''
                pkill -f app.py || true

                echo "Starting Flask app..."
                nohup ./$VENV/bin/python3 app.py > $FLASK_LOG 2>&1 &
                echo $! > flask_pid.txt

                sleep 10

                echo "===== Flask Logs ====="
                cat $FLASK_LOG

                # Check if process is running
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
                pkill socat || true

                echo "Starting socat..."
                nohup socat TCP-LISTEN:$EXPOSE_PORT,fork TCP:localhost:$PORT > $SOCAT_LOG 2>&1 &
                echo $! > socat_pid.txt

                sleep 5

                echo "===== Socat Logs ====="
                cat $SOCAT_LOG
                '''
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                echo "Checking application..."
                for i in {1..10}; do
                    curl -s http://localhost:$EXPOSE_PORT && break
                    echo "Retrying..."
                    sleep 3
                done
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
            echo " Deployment Successful!"
        }

        failure {
            echo "Deployment Failed! Check logs above."
        }
    }
}
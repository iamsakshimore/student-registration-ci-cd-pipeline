pipeline {
    agent any

    environment {
        FLASK_LOG = 'flask.log'
        SOCAT_LOG = 'socat.log'
    }

    stages {

        stage('Clone Repo') {
            steps {
                git branch: 'main', url: 'https://github.com/iamsakshimore/stud-reg-flask-app.git'
            }
        }

        stage('Setup Environment') {
            steps {
                sh '''
                python3 -m venv venv
                ./venv/bin/pip install --upgrade pip
                ./venv/bin/pip install -r requirements.txt
                '''
            }
        }

        stage('Run Flask App') {
            steps {
                sh '''
                pkill -f app.py || true

                nohup ./venv/bin/python3 app.py > $FLASK_LOG 2>&1 &
                echo $! > flask_pid.txt

                sleep 10

                ps -p $(cat flask_pid.txt) || exit 1
                '''
            }
        }

        stage('Expose Port (socat)') {
            steps {
                sh '''
                pkill socat || true

                nohup socat TCP-LISTEN:5050,fork TCP:localhost:5000 > $SOCAT_LOG 2>&1 &
                echo $! > socat_pid.txt

                for i in {1..10}; do
                  ss -tuln | grep 5050 && break
                  sleep 3
                done
                '''
            }
        }

        stage('Test App') {
            steps {
                sh 'curl http://10.10.2.242:5050'
            }
        }
    }

    post {
        always {
            sh 'pkill -f app.py || true'
            sh 'pkill socat || true'
        }
    }
}
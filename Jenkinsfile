pipeline {

    agent any

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out Python Flask project from GitHub...'
            }
        }

        stage('Install Python') {
            steps {
                sh '''
                    sudo yum install -y python3
                '''
            }
        }

        stage('Create Virtual Environment') {
            steps {
                sh '''
                    rm -rf venv

                    python3 -m venv venv

                    ./venv/bin/pip install --upgrade pip
                '''
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    ./venv/bin/pip install -r requirements.txt
                '''
            }
        }

        stage('Test Flask Application') {
            steps {
                sh '''
                    ./venv/bin/python -c "from app import app; print('Flask application loaded successfully')"
                '''
            }
        }

        stage('Deploy to Target Server') {

            environment {

                TARGET_IP = '54.208.147.23'

                CRED_ID = 'ec2-target-key'

                APP_DIR = '~/flask-school-app'
            }

            steps {

                withCredentials([
                    sshUserPrivateKey(
                        credentialsId: "${CRED_ID}",
                        keyFileVariable: 'SSH_KEY',
                        usernameVariable: 'SSH_USER'
                    )
                ]) {

                    sh '''
                        echo "Creating application directory on target server..."

                        ssh -i "$SSH_KEY" \
                        -o StrictHostKeyChecking=no \
                        "$SSH_USER@$TARGET_IP" \
                        "mkdir -p $APP_DIR"
                    '''

                    sh '''
                        echo "Copying Flask application..."

                        scp -i "$SSH_KEY" \
                        -o StrictHostKeyChecking=no \
                        -r app.py requirements.txt templates static \
                        "$SSH_USER@$TARGET_IP:$APP_DIR/"
                    '''

                    sh '''
                        echo "Installing and starting Flask application..."

                        ssh -i "$SSH_KEY" \
                        -o StrictHostKeyChecking=no \
                        "$SSH_USER@$TARGET_IP" "

                            cd $APP_DIR

                            sudo yum install -y python3

                            python3 -m venv venv

                            ./venv/bin/pip install --upgrade pip

                            ./venv/bin/pip install -r requirements.txt

                            pkill -f 'gunicorn.*5000' || true

                            nohup ./venv/bin/gunicorn \
                            --bind 0.0.0.0:5000 \
                            --workers 2 \
                            app:app \
                            > flask.log 2>&1 &

                        "
                    '''
                }
            }
        }
    }
}

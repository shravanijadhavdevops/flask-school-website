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
                echo "Installing Flask dependencies on target server..."

                ssh -i "$SSH_KEY" \
                -o StrictHostKeyChecking=no \
                "$SSH_USER@$TARGET_IP" "
                    cd $APP_DIR

                    sudo yum install -y python3

                    python3 -m venv venv

                    ./venv/bin/pip install --upgrade pip

                    ./venv/bin/pip install -r requirements.txt
                "
            '''

            sh '''
                echo "Starting Flask application with Gunicorn..."

                ssh -i "$SSH_KEY" \
                -o StrictHostKeyChecking=no \
                "$SSH_USER@$TARGET_IP" "
                    cd $APP_DIR

                    nohup ./venv/bin/gunicorn \
                    --bind 0.0.0.0:5000 \
                    --workers 2 \
                    app:app \
                    > flask.log 2>&1 < /dev/null &
                "
            '''

            sh '''
                echo "Checking Flask application..."

                sleep 3

                ssh -i "$SSH_KEY" \
                -o StrictHostKeyChecking=no \
                "$SSH_USER@$TARGET_IP" \
                "curl -I http://localhost:5000"
            '''
        }
    }
}

pipeline {
    agent any

    environment {
        APP_DIR = "/home/ubuntu/flask_web_application"
        PORT = "4000"
        MONGO_URI = credentials('MONGO_URI')
        SECRET_KEY = credentials('SECRET_KEY')
        BRANCH = "master"
        REPO = "https://github.com/sannnn1234/flask_web_application.git"
    }

    stages {

        stage('Clone / Update Code') {
            steps {
                sh '''
                    echo "Fetching latest code..."

                    if [ ! -d "$APP_DIR/.git" ]; then
                        echo "Fresh clone..."
                        rm -rf $APP_DIR
                        git clone -b $BRANCH $REPO $APP_DIR
                    else
                        echo "Updating existing repo..."
                        cd $APP_DIR
                        git fetch --all
                        git reset --hard origin/$BRANCH
                        git clean -fd
                    fi

                    echo "Current commit:"
                    cd $APP_DIR
                    git log -1
                '''
            }
        }

        stage('Verify Latest Files') {
            steps {
                sh '''
                    echo "Checking templates folder..."
                    ls -la $APP_DIR/templates || true
                '''
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    cd $APP_DIR

                    echo "Setting up virtual environment..."

                    if [ ! -d "venv" ]; then
                        python3 -m venv venv
                    fi

                    ./venv/bin/pip install --upgrade pip
                    ./venv/bin/pip install -r requirements.txt
                    ./venv/bin/pip install gunicorn
                '''
            }
        }

        stage('Deploy App') {
            steps {
                sh '''
                    echo "Stopping old app..."
                    pkill -f "gunicorn.*$PORT" || true

                    sleep 2

                    echo "Starting new app..."
                    cd $APP_DIR

                    nohup ./venv/bin/gunicorn -w 2 -b 0.0.0.0:$PORT app:app > app.log 2>&1 &

                    sleep 5

                    echo "Running processes:"
                    ps aux | grep gunicorn
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    echo "Testing app (local)..."
                    curl -f http://localhost:$PORT

                    echo "Testing app (public)..."
                    curl -f http://13.203.223.150:$PORT

                    echo "Last logs:"
                    tail -n 20 $APP_DIR/app.log || true
                '''
            }
        }
    }

    post {
        success {
            echo "Deployment Successful - Latest code is LIVE"
        }
        failure {
            echo "Deployment Failed - Check logs"
        }
    }
}
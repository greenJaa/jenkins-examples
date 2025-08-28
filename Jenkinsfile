pipeline {
    agent any

    environment {
        PATH = "$HOME/.local/bin:$PATH"
    }

    stages {
        stage('Pre-Build') {
            steps {
                echo 'Installing dependencies & checking pre-requisites'
                sh '''
                    apt-get update -y
                    apt-get install -y python3 python3-pip python3-venv binutils curl
                    pip3 install --upgrade pip
                    pip3 install flask pylint pyinstaller
                '''
            }
        }

        stage('Linter') {
            steps {
                echo 'Running pylint'
                sh '''
                    pylint --disable=missing-docstring,invalid-name app.py 
                '''
            }
        }

        stage('Build') {
            steps {
                echo 'Building binary with PyInstaller'
                sh '''
                    pyinstaller --onefile app.py
                '''
            }
        }

        stage('Test') {
            steps {
                echo 'Starting Flask app and testing endpoints'
                sh '''
                    # Start Flask app in background
                    python3 app.py &
                    APP_PID=$!
                    sleep 5

                    # Test root endpoint
                    curl -f http://127.0.0.1:8000/ || (echo "Root endpoint failed" && kill $APP_PID && exit 1)

                    # Test /jenkins endpoint
                    curl -f http://127.0.0.1:8000/jenkins || (echo "/jenkins endpoint failed" && kill $APP_PID && exit 1)

                    kill $APP_PID
                '''
            }
        }

        stage('Archive') {
            steps {
                echo 'Archiving the binary artifact'
                archiveArtifacts artifacts: 'dist/*', onlyIfSuccessful: true
            }
        }
    }
}

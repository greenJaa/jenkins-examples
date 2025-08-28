pipeline {
    agent any

    environment {
        VENV = "${WORKSPACE}/venv"
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/greenJaa/jenkins-examples.git', branch: '03_jenkins_fiel'
            }
        }

        stage('Pre-Build') {
            steps {
                echo 'Installing dependencies & setting up virtualenv'
                sh '''
                    apt-get update -y
                    apt-get install -y python3 python3-pip python3-venv binutils curl

                    # Create venv if not exists
                    if [ ! -d "$VENV" ]; then
                        python3 -m venv $VENV
                    fi

                    # Activate and install requirements
                    . $VENV/bin/activate
                    pip install --upgrade pip
                    pip install flask pylint pyinstaller
                '''
            }
        }

        stage('Linter') {
            steps {
                echo 'Running pylint'
                sh '''
                    . $VENV/bin/activate
                    pylint --disable=missing-docstring,invalid-name app.py
                '''
            }
        }

        stage('Build') {
            steps {
                echo 'Building application with PyInstaller'
                sh '''
                    . $VENV/bin/activate
                    pyinstaller --onefile app.py
                '''
            }
        }

        stage('Test') {
            steps {
                echo 'Testing built binary'
                sh '''
                    . $VENV/bin/activate
                    ./dist/app &
                    APP_PID=$!
                    sleep 3

                    if curl -s http://127.0.0.1:8000 | grep -q 'Hello World'; then
                        echo "Test passed!"
                    else
                        echo "Test failed!"
                        kill $APP_PID
                        exit 1
                    fi

                    kill $APP_PID
                '''
            }
        }

        stage('Archive') {
            steps {
                echo 'Archiving artifacts'
                archiveArtifacts artifacts: 'dist/*', fingerprint: true
            }
        }
    }

    post {
        always {
            echo 'Cleaning up workspace'
        }
        success {
            echo 'Pipeline finished successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}

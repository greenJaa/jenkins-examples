pipeline { 
    agent any

    environment {
        VENV_DIR = "${WORKSPACE}/venv"
    }
 
    triggers {
        githubPush()
    } 
  
    stages { 
        stage('Checkout') {
            steps {
                git branch: '03_jenkins_fiel',
                    url: 'https://github.com/greenJaa/jenkins-examples.git'
            }
        }

        stage('Pre-Build') {
            steps {
                echo 'Installing dependencies & setting up virtualenv'
                sh '''
                    apt-get update -y
                    apt-get install -y python3 python3-pip python3-venv binutils curl
                    if [ ! -d "$VENV_DIR" ]; then
                        python3 -m venv "$VENV_DIR"
                    fi
                    . "$VENV_DIR/bin/activate"
                    pip install --upgrade pip --break-system-packages
                    pip install flask pylint pyinstaller --break-system-packages
                '''
            }
        }

        stage('Linter') {
            steps {
                echo 'Running pylint'
                sh '''
                    . "$VENV_DIR/bin/activate"
                    pylint --disable=missing-docstring,invalid-name app.py
                '''
            }
        }

        stage('Build') {
            steps {
                echo 'Building application with PyInstaller'
                sh '''
                    . "$VENV_DIR/bin/activate"
                    pyinstaller --onefile app.py
                '''
            }
        }

stage('Test') {
    steps {
        echo 'Testing built binary'
        sh '''
            #!/bin/bash
            . "$VENV_DIR/bin/activate"
            APP_PID=0
            ./dist/app &
            APP_PID=$!
            sleep 5

            RESPONSE=$(curl -s http://127.0.0.1:8000 || true)
            echo "Response: $RESPONSE"

if [ "${RESPONSE#*Hello World}" != "$RESPONSE" ]; then
    echo "Test passed!"
    EXIT_CODE=0
            else
                echo "Test failed!"
                EXIT_CODE=1
            fi

            kill $APP_PID || true
            exit $EXIT_CODE
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
            cleanWs()
        }
    }
}

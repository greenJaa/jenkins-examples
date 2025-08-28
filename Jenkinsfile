pipeline {
    agent {
        docker {
            image 'python:3.10-slim'   // Python image with pip
            args '-u root'             // Run as root so we can install packages
        }
    }

    stages {
        stage('Pre-Build') {
            steps {
                echo 'Checking pre-requisites'
                sh '''
                    apt-get update && apt-get install -y curl
                    pip install --no-cache-dir pyinstaller pylint
                    python --version
                    pip --version
                '''
            }
        }

        stage('Linter') {
            steps {
                echo 'Static code analysis check'
                sh '''
                    pylint --disable=missing-docstring,invalid-name app.py
                '''
            }
        }

        stage('Build') {
            steps {
                echo 'Building the Project'
                sh '''
                    pyinstaller --onefile app.py
                '''
            }
        }

        stage('Test') {
            steps {
                echo 'Testing'
                sh '''
                    # Run the built app in the background
                    ./dist/app &

                    # Give it a second to start
                    sleep 2

                    if curl -s http://localhost:8080 > /dev/null; then
                        echo 'POST test: success'
                    else
                        echo 'POST test: fail'
                        exit 1
                    fi

                    if curl -s http://localhost:8080/jenkins > /dev/null; then
                        echo 'POST test with variable: success'
                    else
                        echo 'POST test with variable: fail'
                        exit 1
                    fi
                '''
            }
        }

        stage('Archive') {
            steps {
                echo 'Archiving the artifact'
                archiveArtifacts artifacts: 'dist/*', onlyIfSuccessful: true
            }
        }
    }
}

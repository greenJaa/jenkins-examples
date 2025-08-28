pipeline {
    agent any

    triggers {
        githubPush()   // 🚀 Trigger build on every GitHub push
    }

    stages {
        stage('Pre-Build') {
            steps {
                echo 'Checking pre-requisites'
                sleep(time: 2, unit: 'SECONDS')
                sh '''
                    python3 -m venv venv
                    . venv/bin/activate
                    pip install --upgrade pip --break-system-packages
                    pip install flask pylint pyinstaller --break-system-packages
                '''
            }
        }

        stage('Linter') {
            steps {
                echo 'Running pylint'
                sh '''
                    . venv/bin/activate
                    pylint app.py || true
                '''
            }
        }

        stage('Build') {
            steps {
                echo 'Building with PyInstaller'
                sh '''
                    . venv/bin/activate
                    pyinstaller --onefile app.py
                '''
            }
        }

        stage('Test') {
            steps {
                echo 'Running basic test'
                sh '''
                    . venv/bin/activate
                    python3 -c "import app; print(app.index())"
                '''
            }
        }

        stage('Archive') {
            steps {
                archiveArtifacts artifacts: 'dist/*', fingerprint: true
            }
        }
    }
}

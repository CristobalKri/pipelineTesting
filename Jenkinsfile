pipeline {
    agent any

    environment {
        PROJECT_NAME = "flask-vulnerable-app"
        SONARQUBE_URL = "http://localhost:9000"  // Update with your SonarQube instance URL
        SONARQUBE_TOKEN = "sqa_04ed01f9598ffbf592349e127e7f3c766357eb9c"  // Add your SonarQube token here
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Set Up Environment') {
            steps {
                sh '''
                    python3 -m venv venv
                    . venv/bin/activate
                    pip install --upgrade pip
                    pip install -r requirements.txt
                '''
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'SonarQubeScanner'
                    withSonarQubeEnv('SonarQubeScanner') {
                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                                -Dsonar.projectKey=${PROJECT_NAME} \
                                -Dsonar.sources=. \
                                -Dsonar.host.url=${SONARQUBE_URL} \
                                -Dsonar.login=${SONARQUBE_TOKEN}
                        """
                    }
                }
            }
        }

        stage('Publish Reports') {
            steps {
                waitForQualityGate abortPipeline: true
            }
        }
    }

    post {
        always {
            sh 'rm -rf venv'  // Clean up virtual environment
        }
    }
}

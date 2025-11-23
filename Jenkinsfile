pipeline {
    agent any

    environment {
        PROJECT_NAME = "pipelineTesting"
        SONARQUBE_URL = "http://sonarqube:9000"
        SONARQUBE_TOKEN = "sqa_04ed01f9598ffbf592349e127e7f3c766357eb9c"
        TARGET_URL = "http://172.23.41.49:5000"
    }

    stages {
        stage('Install Python') {
            steps {
                sh '''
                    # Update pacman and install Python and necessary packages
                    sudo pacman -Syu --noconfirm python python-pip python-virtualenv
                '''
            }
        }

        stage('Setup Environment') {
            steps {
                sh '''
                    # Create virtual environment and activate it
                    python3 -m venv venv
                    . venv/bin/activate
                    # Upgrade pip and install dependencies
                    pip install --upgrade pip
                    pip install -r requirements.txt
                '''
            }
        }

        stage('Python Security Audit') {
            steps {
                sh '''
                    . venv/bin/activate
                    pip install pip-audit
                    mkdir -p dependency-check-report
                    # Run pip-audit and generate the report
                    pip-audit -r requirements.txt -f markdown -o dependency-check-report/pip-audit.md || true
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
                                -Dsonar.projectKey=$PROJECT_NAME \
                                -Dsonar.sources=. \
                                -Dsonar.host.url=$SONARQUBE_URL \
                                -Dsonar.login=$SONARQUBE_TOKEN
                        """
                    }
                }
            }
        }

        stage('Dependency Check') {
            environment {
                NVD_API_KEY = credentials('nvdApiKey')
            }
            steps {
                dependencyCheck additionalArguments: "--scan . --format HTML --out dependency-check-report --enableExperimental --enableRetired --nvdApiKey ${NVD_API_KEY}", odcInstallation: 'DependencyCheck'
            }
        }

        stage('Publish Reports') {
            steps {
                publishHTML([
                    allowMissing: false,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: 'dependency-check-report',
                    reportFiles: 'dependency-check-report.html',
                    reportName: 'OWASP Dependency Check Report'
                ])
            }
        }
    }
}

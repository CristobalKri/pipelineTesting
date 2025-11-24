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
                            . venv/bin/activate
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

    //    stage('Dependency Check') {
    //        environment {
    //            NVD_API_KEY = credentials('nvdApiKey')
    //        }
    //        steps {
    //         sh '''
    //                 . venv/bin/activate
    //            dependencyCheck additionalArguments: "--scan . --format HTML --out dependency-check-report --enableExperimental --enableRetired --nvdApiKey ${NVD_API_KEY}", odcInstallation: 'DependencyCheck'
    //         '''     
    //        }
    //    }

    //     stage('Dependency Check') {
    //         steps {
    //         withCredentials([string(credentialsId: 'nvdApiKey', variable: 'NVD_API_KEY')]) {
    //             dependencyCheck additionalArguments: "--scan . --format HTML --out dependency-check-report --enableExperimental --enableRetired --nvdApiKey ${NVD_API_KEY}", odcInstallation: 'DependencyCheck'
    //         }
    //     }
    // }

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

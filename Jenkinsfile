pipeline {
    agent any
    tools {
        nodejs 'nodejs23'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    stages {
        stage('Code Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/pentesterkaran/3-tier-mega-project.git'
            }
        }
        stage('Frontend Compilation Check') {
            steps {
                dir('client') {
                     sh 'find . -name *.js -exec node --check {} +'
                }
            }
        }
        stage('Backend Compilation') {
            steps {
                dir('api') {
                     sh 'find . -name *.js -exec node --check {} +'
                }
            }
        }
        stage('Git Leaks Scan') {
            steps {
                sh 'gitleaks detect --source ./client --exit-code 1'
                sh 'gitleaks detect --source ./api --exit-code 1'
            }
        }
        stage('SoanrQube Scan') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=karanMegaProject \
                     -Dsonar.projectKey=karanMegaProjectKey -Dsonar.sources=.'''
                }
            }
        }
        stage('QualityGates Check') {
            steps {
                timeout(10) {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
            }
        }
        
        
    }
}


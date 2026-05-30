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
        stage('Trivy File System Scan') {
            steps {
                sh 'trivy fs --format table -o fs-report.html .'
            }
        }
        stage('Docker backend image build & Push to DockerHub') {
            steps {
                dir('api') {
                withDockerRegistry(credentialsId: 'dockerhub-keys', url: 'https://index.docker.io/v1/') {
                    sh 'docker build -t deploykarle/devshakops:backend .'
                    sh 'trivy image --format table -o backend-image-report.html deploykarle/devshakops:backend'
                    sh 'docker push deploykarle/devshakops:backend'
                }
            }
            }
        }
        stage('Docker frontend image build & Push to DockerHub') {
            steps {
                dir('client') {
                withDockerRegistry(credentialsId: 'dockerhub-keys', url: 'https://index.docker.io/v1/') {
                    sh 'docker build -t deploykarle/devshakops:frontend .'
                    sh 'trivy image --format table -o frontend-image-report.html deploykarle/devshakops:frontend'
                    sh 'docker push deploykarle/devshakops:frontend'
                }
            }
            }
        }
        stage('Starting Application using docker compose') {
            steps {
                sh 'docker compose down'
                sh 'docker compose up -d --build'
            }
        }
        
    }
}


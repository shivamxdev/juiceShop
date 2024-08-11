pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS = credentials('dockerhub-credentials')
        GITHUB_TOKEN = credentials('github-token')
        ZAP_API_KEY = credentials('zap-api-key')
    }

    options {
        skipDefaultCheckout true
    }
    
    stages {
        stage('Checkout Code') {
            steps {
                checkout([$class: 'GitSCM', 
                          branches: [[name: '*/main']], 
                          userRemoteConfigs: [[url: 'https://github.com/shivamxdev/juiceShop.git']], 
                          shallow: true, 
                          depth: 1])
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm ci'
            }
        }

        stage('Lint') {
            steps {
                sh 'npm run lint'
            }
        }

        stage('Unit Tests') {
            steps {
                sh 'npm test'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("my-app:${env.BUILD_ID}")
                }
            }
        }

        stage('CodeQL Analysis') {
            steps {
                sh '''
                git checkout main
                git checkout -b codeql-analysis
                codeql database create codeql-db
                codeql database analyze codeql-db --format=sarif-latest --output=results.sarif
                '''
            }
        }

        stage('ZAP Scan') {
            steps {
                sh '''
                export ZAP_PATH="/usr/share/zaproxy"
                export ZAP_PORT=8090
                sh ./start_zap.sh $ZAP_PATH $ZAP_PORT $ZAP_API_KEY
                zap-cli status -t 120
                zap-cli active-scan http://localhost
                zap-cli report -o zap_report.html -f html
                '''
            }
        }

        stage('Rebase') {
            steps {
                sh '''
                git fetch origin
                git rebase origin/main
                git push origin HEAD:main --force
                '''
            }
        }

        stage('Release') {
            steps {
                script {
                    def version = sh(script: "npm version --no-git-tag-version patch", returnStdout: true).trim()
                    git tag "v${version}"
                    git push origin --tags
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                kubectl apply -f k8s/deployment.yaml
                kubectl rollout status deployment/my-app
                '''
            }
        }
    }

    post {
        always {
            cleanWs()
        }
        success {
            echo 'Pipeline completed successfully.'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}

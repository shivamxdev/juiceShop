pipeline {
    agent none
    environment {
        ANGULAR_CLI_VERSION = '13'
    }
    stages {
        stage('Lint') {
            agent { label 'linux' }
            steps {
                git url: 'https://github.com/shivamxdev/juiceShop.git'
                nodejs('NodeJS 18') {
                    sh 'npm install -g @angular/cli@$ANGULAR_CLI_VERSION'
                    sh '''
                        npm install --ignore-scripts
                        cd frontend
                        npm install --ignore-scripts --legacy-peer-deps
                    '''
                    sh 'npm run lint'
                    sh '''
                        npm run lint:config -- -f ./config/7ms.yml &&
                        npm run lint:config -- -f ./config/addo.yml &&
                        npm run lint:config -- -f ./config/bodgeit.yml &&
                        npm run lint:config -- -f ./config/ctf.yml &&
                        npm run lint:config -- -f ./config/default.yml &&
                        npm run lint:config -- -f ./config/fbctf.yml &&
                        npm run lint:config -- -f ./config/juicebox.yml &&
                        npm run lint:config -- -f ./config/mozilla.yml &&
                        npm run lint:config -- -f ./config/oss.yml &&
                        npm run lint:config -- -f ./config/quiet.yml &&
                        npm run lint:config -- -f ./config/tutorial.yml &&
                        npm run lint:config -- -f ./config/unsafe.yml
                    '''
                }
            }
        }
        stage('Coding Challenge RSN') {
            agent { label 'windows' }
            steps {
                git url: 'https://github.com/shivamxdev/juiceShop.git'
                nodejs('NodeJS 18') {
                    sh 'npm install -g @angular/cli@$ANGULAR_CLI_VERSION'
                    sh 'npm install'
                    sh 'npm run rsn'
                }
            }
        }
        stage('Test') {
            matrix {
                axes {
                    axis {
                        name 'OS'
                        values 'linux', 'windows', 'mac'
                    }
                    axis {
                        name 'NODE_VERSION'
                        values '16', '18', '20'
                    }
                }
                stages {
                    stage('Checkout') {
                        agent { label "${OS}" }
                        steps {
                            git url: 'https://github.com/shivamxdev/juiceShop.git'
                        }
                    }
                    stage('Install NodeJS and CLI tools') {
                        agent { label "${OS}" }
                        steps {
                            nodejs("NodeJS ${NODE_VERSION}") {
                                sh 'npm install -g @angular/cli@$ANGULAR_CLI_VERSION'
                                sh 'npm install'
                            }
                        }
                    }
                    stage('Unit Tests') {
                        agent { label "${OS}" }
                        steps {
                            script {
                                retry(3) {
                                    timeout(time: 15, unit: 'MINUTES') {
                                        sh 'npm test'
                                    }
                                }
                            }
                        }
                    }
                    stage('Upload Coverage') {
                        when {
                            allOf {
                                expression { return env.BRANCH_NAME == 'master' }
                                expression { return OS == 'linux' }
                                expression { return NODE_VERSION == '16' }
                            }
                        }
                        steps {
                            archiveArtifacts artifacts: 'build/reports/coverage/**/*.info', allowEmptyArchive: true
                        }
                    }
                }
            }
        }
        stage('API Test') {
            matrix {
                axes {
                    axis {
                        name 'OS'
                        values 'linux', 'windows', 'mac'
                    }
                    axis {
                        name 'NODE_VERSION'
                        values '16', '18', '20'
                    }
                }
                stages {
                    stage('Checkout') {
                        agent { label "${OS}" }
                        steps {
                            git url: 'https://github.com/shivamxdev/juiceShop.git'
                        }
                    }
                    stage('Install NodeJS and CLI tools') {
                        agent { label "${OS}" }
                        steps {
                            nodejs("NodeJS ${NODE_VERSION}") {
                                sh 'npm install -g @angular/cli@$ANGULAR_CLI_VERSION'
                                sh 'npm install'
                            }
                        }
                    }
                    stage('Integration Tests') {
                        agent { label "${OS}" }
                        steps {
                            script {
                                retry(3) {
                                    timeout(time: 5, unit: 'MINUTES') {
                                        sh '''
                                            if [ "$RUNNER_OS" == "Windows" ]; then
                                            set NODE_ENV=test
                                            else
                                            export NODE_ENV=test
                                            fi
                                            npm run frisby
                                        '''
                                    }
                                }
                            }
                        }
                    }
                    stage('Upload Coverage') {
                        when {
                            allOf {
                                expression { return env.BRANCH_NAME == 'master' }
                                expression { return OS == 'linux' }
                                expression { return NODE_VERSION == '16' }
                            }
                        }
                        steps {
                            archiveArtifacts artifacts: 'build/reports/coverage/**/*.info', allowEmptyArchive: true
                        }
                    }
                }
            }
        }
        stage('Coverage Report') {
            agent { label 'linux' }
            when {
                allOf {
                    expression { return env.BRANCH_NAME == 'master' }
                }
            }
            steps {
                git url: 'https://github.com/shivamxdev/juiceShop.git'
                // Steps to download and publish coverage report
                // Similar to what is in ci.yml file for this stage
            }
        }
        // Add other stages based on ci.yml
    }
    post {
        always {
            cleanWs()
        }
    }
}

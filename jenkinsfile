pipeline {
    agent any

    environment {
        IMAGE_NAME = 'siku9786/todo-app'
        IMAGE_TAG = "${BUILD_NUMBER}"
        NODE_IMAGE = 'node:18'
        SONAR_SCANNER_IMAGE = 'sonarsource/sonar-scanner-cli:latest'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/sikander-riaz/DevOps-assignment-1'
            }
        }

        stage('Build & Test') {
            parallel {
                stage('Install') {
                    steps {
                        script {
                            docker.image("${NODE_IMAGE}").inside {
                                sh 'npm install'
                            }
                        }
                    }
                }
                stage('Test') {
                    steps {
                        script {
                            docker.image("${NODE_IMAGE}").inside {
                                sh 'npm test -- --coverage'
                            }
                        }
                    }
                }
            }
        }

        stage('SonarQube Scan') {
            steps {
                script {
                    docker.image("${SONAR_SCANNER_IMAGE}").inside {
                        withSonarQubeEnv('sonarqube') {
                            withCredentials([string(credentialsId: 'gen-token', variable: 'SONAR_TOKEN')]) {
                                sh '''
                                    sonar-scanner \
                                      -Dsonar.projectKey=todo \
                                      -Dsonar.sources=. \
                                      -Dsonar.javascript.lcov.reportPaths=coverage/lcov.info \
                                      -Dsonar.login=${SONAR_TOKEN}
                                '''
                            }
                        }
                    }
                }
            }
            post {
                always {
                    echo 'SonarQube analysis complete — check dashboard for Quality Gate results.'
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh """
                  docker build -t ${IMAGE_NAME}:${IMAGE_TAG} -t ${IMAGE_NAME}:latest .
                """
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([string(credentialsId: 'docker-hub-token', variable: 'DOCKER_TOKEN')]) {
                    sh '''
                        echo "${DOCKER_TOKEN}" | docker login -u siku9786 --password-stdin
                        docker push ${IMAGE_NAME}:${IMAGE_TAG}
                        docker push ${IMAGE_NAME}:latest
                    '''
                }
            }
        }
    }

    post {
        always {
            sh 'docker system prune -f || true'
            echo "Pipeline finished successfully "
        }
    }
}

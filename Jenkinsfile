pipeline {
    agent any

    options {
        buildDiscarder logRotator(
            daysToKeepStr: '30',
            numToKeepStr: '5'
        )
        timestamps()
    }

    tools {
        nodejs 'NodeJS'
    }

    environment {

        // DockerHub Credentials
        DOCKERHUB_CREDS = credentials('docker-hub-credentials')

        // Image Info
        IMAGE_TAG = "14"
        FRONTEND_IMAGE = "fekry34455/frontend-reddit-1"

        // SonarQube
        SONAR_PROJECT_KEY  = "frontend-second"
        SONAR_PROJECT_NAME = "Frontend_second_app"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scmGit(
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[
                        url: 'https://github.com/mohamedmostafa33/nti-final-project-ci'
                    ]]
                )
            }
        }

        stage('Install Frontend Dependencies') {
            steps {
                dir('src/frontend') {
                    sh 'npm install'
                }
            }
        }

        stage('Run Tests') {
            steps {
                dir('src/frontend') {
                    sh 'npm test || true'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    script {
                        def scannerHome = tool 'sonar-scanner'
                        sh """
                        ${scannerHome}/bin/sonar-scanner \
                        -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                        -Dsonar.projectName=${SONAR_PROJECT_NAME} \
                        -Dsonar.sources=src/frontend \
                        -Dsonar.language=js
                        """
                    }
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh "docker build -t frontend:${IMAGE_TAG} ./src/frontend"
            }
        }

        stage('Trivy Scan') {
            steps {
                sh "trivy image --severity HIGH,CRITICAL frontend:${IMAGE_TAG} || true"
            }
        }

        stage('Push Image to DockerHub') {
            steps {
                script {
                    docker.withRegistry('', 'docker-hub-credentials') {

                        sh "docker tag frontend:${IMAGE_TAG} ${FRONTEND_IMAGE}:${IMAGE_TAG}"
                        sh "docker push ${FRONTEND_IMAGE}:${IMAGE_TAG}"

                        sh "docker tag frontend:${IMAGE_TAG} ${FRONTEND_IMAGE}:latest"
                        sh "docker push ${FRONTEND_IMAGE}:latest"
                    }
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline finished"
        }
        success {
            echo "Deployment Successful 🚀"
        }
        failure {
            echo "Pipeline Failed ❌"
        }
    }
}

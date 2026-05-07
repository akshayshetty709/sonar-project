pipeline {
    agent any

    environment {
        // Harbor Details
        HARBOR_REGISTRY = "harbor.yourdomain.com" // Replace with your Harbor URL
        HARBOR_PROJECT  = "my-project"            // Replace with your Harbor Project name
        IMAGE_NAME      = "gatewayservice"
        IMAGE_TAG       = "latest"
        
        // Full Harbor Repo Path
        HARBOR_REPO     = "${HARBOR_REGISTRY}/${HARBOR_PROJECT}/${IMAGE_NAME}"
        
        // SonarScanner Tool
        SCANNER_HOME    = tool 'SonarScanner'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/akshayshetty709/crm-backend.git'
            }
        }

        stage('SonarQube Scan') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh """
                    ${SCANNER_HOME}/bin/sonar-scanner \
                    -Dsonar.projectKey=gatewayservice \
                    -Dsonar.sources=.
                    """
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                // Building the image locally
                sh "docker build -t ${IMAGE_NAME}:latest ./gatewayservice"
            }
        }

        stage('Login to Harbor') {
            steps {
                // Ensure 'harbor-credentials' is created in Jenkins as Username/Password type
                withCredentials([usernamePassword(credentialsId: 'harbor-credentials', 
                                                 passwordVariable: 'HARBOR_PW', 
                                                 usernameVariable: 'HARBOR_USER')]) {
                    sh "echo ${HARBOR_PW} | docker login ${HARBOR_REGISTRY} -u ${HARBOR_USER} --password-stdin"
                }
            }
        }

        stage('Tag for Harbor') {
            steps {
                sh "docker tag ${IMAGE_NAME}:latest ${HARBOR_REPO}:${IMAGE_TAG}"
            }
        }

        stage('Push to Harbor') {
            steps {
                sh "docker push ${HARBOR_REPO}:${IMAGE_TAG}"
            }
        }

        stage('Cleanup Local Images') {
            steps {
                sh """
                docker rmi ${IMAGE_NAME}:latest || true
                docker rmi ${HARBOR_REPO}:${IMAGE_TAG} || true
                """
            }
        }
    }
    
    post {
        always {
            sh "docker logout ${HARBOR_REGISTRY}"
        }
    }
}

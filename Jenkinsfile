pipeline {
    agent { label 'agent-1' }
    
    environment {
        REPO_URL    = 'https://github.com/satyamsingh24/jenkins-versioning.git'
        IMAGE_NAME  = '176583374037.dkr.ecr.ap-south-1.amazonaws.com/myapp-repo'
        AWS_REGION  = 'ap-south-1'
    }
   
   triggers {
        pollSCM('H/2 * * * *')  // har 2 min check karo
    }
    
    stages {
        stage('Checkout') {
            steps {
                git credentialsId: 'github-creds',
                    url: "${REPO_URL}",
                    branch: 'main'
            }
        }
        
        stage('Generate Version') {
            steps {
                script {
                    // Version banao: v1.0.BUILD_NUMBER
                    env.VERSION = "v1.0.${BUILD_NUMBER}"
                    echo "Version: ${env.VERSION}"
                }
            }
        }
        
        stage('Build Docker Image') {
            steps {
                sh """
                    docker build -t ${IMAGE_NAME}:${VERSION} .
                    docker tag ${IMAGE_NAME}:${VERSION} ${IMAGE_NAME}:latest
                """
            }
        }
        
        stage('Push to ECR') {
            steps {
                sh """
                    aws ecr get-login-password --region ${AWS_REGION} | \
                    docker login --username AWS \
                    --password-stdin ${IMAGE_NAME}
                    
                    docker push ${IMAGE_NAME}:${VERSION}
                    docker push ${IMAGE_NAME}:latest
                """
            }
        }
        
        stage('Git Tag') {
    steps {
        withCredentials([usernamePassword(
            credentialsId: 'github-creds',
            usernameVariable: 'GIT_USER',
            passwordVariable: 'GIT_TOKEN'
        )]) {
            sh """
                git config user.email "jenkins@example.com"
                git config user.name "Jenkins"
                git tag ${VERSION}
                git push https://${GIT_USER}:${GIT_TOKEN}@github.com/satyamsingh24/jenkins-versioning.git ${VERSION}
            """
        }
    }
}
        
        stage('Verify') {
            steps {
                sh """
                    echo "✅ Version: ${VERSION}"
                    echo "✅ Image pushed: ${IMAGE_NAME}:${VERSION}"
                    git tag -l
                """
            }
        }
    }
    
    post {
        success {
            echo "🎉 Build ${VERSION} Successfully Tagged!"
        }
        failure {
            echo "❌ Build Failed!"
        }
    }
}

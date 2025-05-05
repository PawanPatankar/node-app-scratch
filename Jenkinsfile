pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        ECR_REGISTRY = '448049808628.dkr.ecr.us-east-1.amazonaws.com'
        ECR_REPO = 'node-app'
        IMAGE_TAG = "${BUILD_NUMBER}"
    }


    stages {
        stage('Checkout Code') {
            steps {
                git url: 'https://github.com/PawanPatankar/node-app-scratch.git', branch: 'main'
            }
        }

        stage('Install Dependencies') {
            steps {
                    sh 'npm install'  
            }
        }

        stage('Run Tests') {
            steps {
                    sh 'npm test || echo "No tests found, would add tests in a real project" '
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh """
                        docker build -t $ECR_REPO:$IMAGE_TAG .
                        docker tag $ECR_REPO:$IMAGE_TAG $ECR_REGISTRY/$ECR_REPO:$IMAGE_TAG
                    """
                }
            }
        }

        stage('Push to ECR') {
            steps {
                script {
                    sh """
                        aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REGISTRY
                        docker push $ECR_REGISTRY/$ECR_REPO:$IMAGE_TAG
                    """
                }
            }
        }
        stage('Checkout Code') {
            steps {
                git credentialsId: 'GITHUB', url: 'https://github.com/PawanPatankar/node-app-scratch.git'
            }
        }
        stage('Update Deployment file') {
            environment {
                GIT_REPO_NAME = "node-app-scratch"
                GIT_USER_NAME = "PawanPatankar
            }
            steps {
                dir('Kubernetes') {
                    withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                        sh '''
                            git config user.email "pp123@gmail.com"
                            git config user.name "PawanPatankar"
                            BUILD_NUMBER=${BUILD_NUMBER}
                            echo $BUILD_NUMBER
                            sed -i "s/replaceImageTag/${BUILD_NUMBER}/g" kubernetes/deployment.yaml"
                            git add kubernetes/deployment.yaml
                            git commit -m "Update deployment image to version ${BUILD_NUMBER}"
                            git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                        '''
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment successful!'
        }
        failure {
            echo 'Something went wrong...'
        }
    }
}
    
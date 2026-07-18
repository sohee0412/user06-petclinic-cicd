pipeline {
    agent any

    environment {
        AWS_REGION     = 'us-west-1'
        ECR_REPO       = '450444046629.dkr.ecr.us-west-1.amazonaws.com/user06-petclinic'
        IMAGE_TAG_V    = "v${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    credentialsId: 'github-cred',
                    url: 'https://github.com/sohee0412/user06-petclinic-cicd.git'
            }
        }

        stage('Build') {
            steps {
                sh 'chmod +x mvnw || true'
                sh './mvnw clean package -DskipTests || mvn clean package -DskipTests'
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    sh """
                        docker build -t ${ECR_REPO}:${IMAGE_TAG_V} -t ${ECR_REPO}:${GIT_COMMIT} .
                    """
                }
            }
        }

        stage('ECR Login & Push') {
            steps {
                sh """
                    LOGIN_PASSWORD=\$(docker run --rm amazon/aws-cli ecr get-login-password --region ${AWS_REGION})
                    echo \$LOGIN_PASSWORD | docker login --username AWS --password-stdin ${ECR_REPO}

                    docker push ${ECR_REPO}:${IMAGE_TAG_V}
                    docker push ${ECR_REPO}:${GIT_COMMIT}
                """
            }
        }
    }

    post {
        success {
            echo "Build and push succeeded: ${ECR_REPO}:${IMAGE_TAG_V}"
        }
        failure {
            echo "Pipeline failed. Check console output."
        }
    }
}


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

        stage('SonarQube Analysis') {
            steps {
                withCredentials([string(credentialsId: 'sonarqube-token', variable: 'SONAR_TOKEN')]) {
                    sh """
                        ./mvnw sonar:sonar \
                          -Dsonar.host.url=https://10.6.0.43:9000 \
                          -Dsonar.login=\$SONAR_TOKEN \
                          -Dsonar.projectKey=user06-petclinic
                    """
                }
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

        stage('Trivy Scan') {
            steps {
                sh """
                    docker run --rm \
                      -v /var/run/docker.sock:/var/run/docker.sock \
                      -v \$HOME/.cache/trivy:/root/.cache/ \
                      aquasec/trivy:latest image \
                      --severity HIGH,CRITICAL \
                      --exit-code 0 \
                      ${ECR_REPO}:${IMAGE_TAG_V}
                """
            }
        }

        stage('ECR Login & Push') {
            steps {
                withCredentials([
                    string(credentialsId: 'aws-access-key-id', variable: 'AWS_ACCESS_KEY_ID'),
                    string(credentialsId: 'aws-key', variable: 'AWS_SECRET_ACCESS_KEY')
                ]) {
                    sh """
                        LOGIN_PASSWORD=\$(docker run --rm \
                          -e AWS_ACCESS_KEY_ID=\$AWS_ACCESS_KEY_ID \
                          -e AWS_SECRET_ACCESS_KEY=\$AWS_SECRET_ACCESS_KEY \
                          amazon/aws-cli ecr get-login-password --region ${AWS_REGION})
                        echo \$LOGIN_PASSWORD | docker login --username AWS --password-stdin ${ECR_REPO}

                        docker push ${ECR_REPO}:${IMAGE_TAG_V}
                        docker push ${ECR_REPO}:${GIT_COMMIT}
                    """
                }
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


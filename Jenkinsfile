pipeline {
    agent any

    parameters {
        string(name: 'VERSION', defaultValue: 'latest', description: 'Docker image version')
    }

    environment {
        AWS_REGION = 'us-east-1'
        AWS_ACCOUNT_ID = 'your_aws_account_id'
        ECR_REGISTRY = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
        USER_SERVICE_REPO = "${ECR_REGISTRY}/user-service"
        PRODUCT_SERVICE_REPO = "${ECR_REGISTRY}/product-service"
        ORDER_SERVICE_REPO = "${ECR_REGISTRY}/order-service"
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/mansuralamkhan/microservice.git'
            }
        }

        stage('Login to AWS ECR') {
            steps {
                script {
                    sh "aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY}"
                }
            }
        }

        stage('Build and Push Docker Images') {
            parallel {
                stage('User Service') {
                    steps {
                        script {
                            buildAndPushDockerImage('user-service', 'user-service/Dockerfile', USER_SERVICE_REPO)
                        }
                    }
                }
                stage('Product Service') {
                    steps {
                        script {
                            buildAndPushDockerImage('product-service', 'product-service/Dockerfile', PRODUCT_SERVICE_REPO)
                        }
                    }
                }
                stage('Order Service') {
                    steps {
                        script {
                            buildAndPushDockerImage('order-service', 'order-service/Dockerfile', ORDER_SERVICE_REPO)
                        }
                    }
                }
            }
        }

        // stage('Notify') {
        //     steps {
        //         script {
        //             if (currentBuild.result == 'SUCCESS') {
        //                 slackSend(channel: '#deployments', message: "Docker images successfully pushed: ${env.JOB_NAME} ${env.BUILD_NUMBER}")
        //             } else {
        //                 slackSend(channel: '#deployments', message: "Docker image push failed: ${env.JOB_NAME} ${env.BUILD_NUMBER}")
        //             }
        //         }
        //     }
        // }
    }

    post {
        always {
            script {
                sh "docker rmi ${USER_SERVICE_REPO}:${params.VERSION}"
                sh "docker rmi ${USER_SERVICE_REPO}:latest"
                sh "docker rmi ${PRODUCT_SERVICE_REPO}:${params.VERSION}"
                sh "docker rmi ${PRODUCT_SERVICE_REPO}:latest"
                sh "docker rmi ${ORDER_SERVICE_REPO}:${params.VERSION}"
                sh "docker rmi ${ORDER_SERVICE_REPO}:latest"
            }
        }
    }
}

def buildAndPushDockerImage(serviceName, dockerfilePath, repository) {
    docker.withRegistry("https://${ECR_REGISTRY}") {
        def image = docker.build("${repository}:${params.VERSION}", "-f ${dockerfilePath} .")
        image.push()
        image.push('latest')
    }
}

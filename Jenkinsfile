pipeline {
    agent any

    tools {
        maven "Maven 3.8.4"
    }

    environment {
        REGION = "us-east-1"
        AWS_USER_ID = credentials("jenkins-aws-user-id")
        MICROSERVICE = "user_microservice"
    }
    parameters {
        string(name: "BRANCH_NAME", description: "The branch to checkout.")
    }

    stages {
        stage("Checkout") {
            steps {
                git(branch:"${params.BRANCH_NAME}", url:"https://github.com/The-Black-Eyed-Beans/aline-user-microservice-kwa.git")
                sh "git submodule init && git submodule update"
            }
        }

        stage("Compiling") {
            steps {
                sh "mvn clean compile"
            }
        }

        stage("Testing") {
            steps {
                sh "echo testing"
            }
        }

        stage("Packaging") {
            steps {
                sh "mvn clean package -Dmaven.test.skip=true"
            }
        }

        stage("Building Image") {
            steps {
                sh "docker build -t kwa-${MICROSERVICE} ."
            }
        }

        stage("Deploying") {
            steps {
                withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'aws-key', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY']]) {
    
                    sh "aws ecr get-login-password --region ${REGION} | docker login --username AWS --password-stdin ${AWS_USER_ID}.dkr.ecr.${REGION}.amazonaws.com"
                    sh "docker tag kwa-${MICROSERVICE}:latest ${AWS_USER_ID}.dkr.ecr.${REGION}.amazonaws.com/kwa-${MICROSERVICE}:latest"
                    sh "docker push ${AWS_USER_ID}.dkr.ecr.${REGION}.amazonaws.com/kwa-${MICROSERVICE}:latest"
                
                }
            }
        }
    }

    post {
        always {
            echo "pipeline done"
        }

        cleanup {
            sh "docker rmi ${AWS_USER_ID}.dkr.ecr.${REGION}.amazonaws.com/kwa-${MICROSERVICE}:latest"
        }
    }
}

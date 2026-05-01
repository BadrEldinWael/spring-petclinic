pipeline {
    agent any 

    tools {
        maven 'Maven-3.9'
        jdk 'JDK-17'
    }

    environment {
        AWS_REGION = 'us-east-1'
        ECR_REPO = 'spring-petclinic'
        AWS_ACCOUNT_ID = '200098097766'   
        IMAGE_TAG = 'latest'
    }

    stages {

        stage('clone') {
            steps {
                git branch: 'main',
                url: 'https://github.com/spring-projects/spring-petclinic.git'
            }
        }

        stage('change config') {
            steps {
                sh 'echo server.port=8081 >> src/main/resources/application.properties'
            }
        }

        stage('build jar') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('build docker image') {
            steps {
                sh 'docker build -t $ECR_REPO:$IMAGE_TAG .'
            }
        }

        stage('login to ECR') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-creds'
                ]]) {
                    sh '''
                    aws ecr get-login-password --region $AWS_REGION | \
                    docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com
                    '''
                }
            }
        }

        stage('tag image') {
            steps {
                sh '''
                docker tag $ECR_REPO:$IMAGE_TAG \
                $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_REPO:$IMAGE_TAG
                '''
            }
        }

        stage('push to ECR') {
            steps {
                sh '''
                docker push $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_REPO:$IMAGE_TAG
                '''
            }
        }

        stage('run container') {
            steps {
                sh '''
                docker stop petclinic || true
                docker rm petclinic || true
                docker run -d -p 8081:8081 --name petclinic \
                $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_REPO:$IMAGE_TAG
                '''
            }
        }
    }
}

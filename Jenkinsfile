pipeline {
    agent any
    environment {
        COMMIT_HASH = "${sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()}"
        SERVICE_NAME = "eureka-server"
        ECR_REGISTRY_URI = "135316859264.dkr.ecr.us-east-2.amazonaws.com"
    }
    stages {
        stage('Clean and test target') {
            steps {
                sh 'mvn clean test'
            }
        }
        stage('Test and Package') {
            steps {
                sh 'mvn package'
            }
        }
  //      stage('Quality gate') {
 //           steps {
//                waitForQualityGate abortPipeline: true
//            }
//        }
        stage('Docker Build') {
            steps {
                echo 'Deploying....'
                sh "aws ecr get-login-password --region us-east-2 | docker login --username AWS --password-stdin ${ECR_REGISTRY_URI}"
                sh "docker build -t ${SERVICE_NAME}:${COMMIT_HASH} ."
                sh "docker tag ${SERVICE_NAME}:${COMMIT_HASH} ${ECR_REGISTRY_URI}/${SERVICE_NAME}:${COMMIT_HASH}"
                sh "docker push ${ECR_REGISTRY_URI}/${SERVICE_NAME}:${COMMIT_HASH}"
            }
        }
            stage('Deploy') {
              steps {
                echo 'Deploying cloudformation..'
                sh "aws cloudformation deploy --stack-name ${SERVICE_NAME}-stack --template-file ./orchestratorECS.yml  --parameter-overrides ApplicationName=${SERVICE_NAME} ECRepositoryUri=${ECR_REGISTRY_URI}/${SERVICE_NAME}:${COMMIT_HASH} --capabilities CAPABILITY_IAM CAPABILITY_NAMED_IAM --region us-east-2"
              }
            }
    }
    post {
        always {
            sh 'mvn clean'
            sh 'docker system prune -af'
        }
    }
}

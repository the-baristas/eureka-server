pipeline {
    agent any
    environment {
        COMMIT_HASH = "${sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()}"
        AWS_ID = "135316859264"
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
                sh "aws ecr get-login-password --region us-east-2 | docker login --username AWS --password-stdin 135316859264.dkr.ecr.us-east-2.amazonaws.com"
                sh "docker build -t eureka-server:latest ."
                 sh "docker tag eureka-server:latest 135316859264.dkr.ecr.us-east-2.amazonaws.com/eureka-server:latest"
                 sh "docker push 135316859264.dkr.ecr.us-east-2.amazonaws.com/eureka-server:latest"
            }
        }
            stage('Deploy') {
              steps {
                
                echo 'Deploying cloudformation..'
                sh "aws cloudformation deploy --stack-name EurekaServer --template-file ./eurekaServerECS.yml --capabilities CAPABILITY_IAM CAPABILITY_NAMED_IAM --region us-east-2"
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

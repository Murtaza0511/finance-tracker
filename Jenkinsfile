pipeline {
  agent any

  environment {
    AWS_REGION      = 'us-east-1'
    AWS_ACCOUNT_ID  = credentials('aws-account-id')
    ECR_REGISTRY    = "${AWS_ACCOUNT_ID}.dkr.ecr.us-east-1.amazonaws.com"
    BACKEND_IMAGE   = "${ECR_REGISTRY}/finance-backend"
    FRONTEND_IMAGE  = "${ECR_REGISTRY}/finance-frontend"
    IMAGE_TAG       = "${env.GIT_COMMIT[0..6]}"
    S3_BUCKET       = 'finance-frontend-YOUR-NAME'
    ECS_CLUSTER     = 'finance-cluster'
    ECS_SERVICE     = 'finance-backend-service'
  }

  stages {

    stage('Checkout') {
      steps {
        echo 'Pulling latest code from GitHub...'
        checkout scm
      }
    }

    stage('Install & Test Backend') {
      steps {
        echo 'Installing backend dependencies and running tests...'
        sh '''
          cd backend
          npm install
          npm test -- --passWithNoTests
        '''
      }
    }

    stage('Install & Test Frontend') {
      steps {
        echo 'Installing frontend dependencies...'
        sh '''
          cd frontend
          npm install
          npm test -- --watchAll=false --passWithNoTests
        '''
      }
    }

    stage('Build Docker Images') {
      steps {
        echo 'Building Docker images...'
        sh 'docker build -t ${BACKEND_IMAGE}:${IMAGE_TAG} -t ${BACKEND_IMAGE}:latest ./backend'
        sh 'docker build -t ${FRONTEND_IMAGE}:${IMAGE_TAG} -t ${FRONTEND_IMAGE}:latest ./frontend'
      }
    }

    stage('Push to AWS ECR') {
      steps {
        echo 'Logging into AWS ECR and pushing images...'
        withAWS(credentials: 'aws-creds', region: "${AWS_REGION}") {
          sh 'aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY}'
          sh 'docker push ${BACKEND_IMAGE}:${IMAGE_TAG}'
          sh 'docker push ${BACKEND_IMAGE}:latest'
        }
      }
    }

    stage('Deploy Backend to ECS') {
      steps {
        echo 'Deploying new version to ECS...'
        withAWS(credentials: 'aws-creds', region: "${AWS_REGION}") {
          sh '''
            aws ecs update-service \
              --cluster ${ECS_CLUSTER} \
              --service ${ECS_SERVICE} \
              --force-new-deployment \
              --region ${AWS_REGION}
          '''
        }
      }
    }

    stage('Deploy Frontend to S3') {
      steps {
        echo 'Building React app and uploading to S3...'
        sh 'cd frontend && npm run build'
        withAWS(credentials: 'aws-creds', region: "${AWS_REGION}") {
          sh 'aws s3 sync frontend/build/ s3://${S3_BUCKET}/ --delete'
        }
      }
    }

    stage('Verify Deployment') {
      steps {
        echo 'Waiting for ECS to stabilize...'
        withAWS(credentials: 'aws-creds', region: "${AWS_REGION}") {
          sh '''
            aws ecs wait services-stable \
              --cluster ${ECS_CLUSTER} \
              --services ${ECS_SERVICE}
            echo "Deployment verified!"
          '''
        }
      }
    }
  }

  post {
    success {
      echo "Pipeline SUCCESS - Build ${IMAGE_TAG} deployed!"
    }
    failure {
      echo "Pipeline FAILED - Check the stage logs above"
    }
  }
}
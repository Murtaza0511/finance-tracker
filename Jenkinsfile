pipeline {
agent any

environment {
AWS_REGION = 'us-east-1'
AWS_ACCOUNT_ID = '251819893831'

```
ECR_REGISTRY = "${AWS_ACCOUNT_ID}.dkr.ecr.us-east-1.amazonaws.com"

BACKEND_IMAGE = "${ECR_REGISTRY}/finance-backend"
FRONTEND_IMAGE = "${ECR_REGISTRY}/finance-frontend"

IMAGE_TAG = "${BUILD_NUMBER}"
```

}

stages {

```
stage('Checkout') {
  steps {
    git branch: 'main',
    url: 'https://github.com/Murtaza0511/finance-tracker.git'
  }
}

stage('Build Backend') {
  steps {
    dir('backend') {
      sh 'npm install'
    }
  }
}

stage('Build Frontend') {
  steps {
    dir('frontend') {
      sh 'npm install'
    }
  }
}

stage('Docker Build Backend') {
  steps {
    sh 'docker build -t ${BACKEND_IMAGE}:${IMAGE_TAG} ./backend'
  }
}

stage('Docker Build Frontend') {
  steps {
    sh 'docker build -t ${FRONTEND_IMAGE}:${IMAGE_TAG} ./frontend'
  }
}

stage('Push Images To ECR') {
  steps {
    withAWS(credentials: 'aws-creds', region: AWS_REGION) {

      sh '''
        aws ecr get-login-password --region us-east-1 | \
        docker login --username AWS --password-stdin ${ECR_REGISTRY}
      '''

      sh 'docker push ${BACKEND_IMAGE}:${IMAGE_TAG}'
      sh 'docker push ${FRONTEND_IMAGE}:${IMAGE_TAG}'
    }
  }
}
```

}

post {
success {
echo 'Pipeline completed successfully!'
}

```
failure {
  echo 'Pipeline failed!'
}
```

}
}

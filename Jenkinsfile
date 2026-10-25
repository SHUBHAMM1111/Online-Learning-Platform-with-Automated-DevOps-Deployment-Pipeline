pipeline {
  agent any
  environment {
    IMAGE = "shubhammane11/learning-app"
    SERVERS = "98.84.172.245 13.218.219.151"
  }
  stages {
    stage('Checkout') { steps { checkout scm } }
    stage('Validate') { steps { sh 'test -f index.html && echo "Validation OK"' } }
    stage('Docker Build') { steps { sh 'docker build -t $IMAGE:$BUILD_NUMBER -t $IMAGE:latest .' } }
    stage('Push') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'U', passwordVariable: 'P')]) {
          sh 'echo $P | docker login -u $U --password-stdin'
          sh 'docker push $IMAGE:$BUILD_NUMBER && docker push $IMAGE:latest'
        }
      }
    }
    stage('Deploy') {
      steps {
        sh '''
        for s in $SERVERS; do
          ssh -o StrictHostKeyChecking=no ubuntu@$s "docker pull $IMAGE:latest && docker rm -f learning-app || true; docker run -d --name learning-app --restart always -p 3000:80 -e DB_HOST=10.0.1.195 -e DB_NAME=learningdb -e DB_USER=learner -e DB_PASSWORD=learnpass $IMAGE:latest"
        done
        '''
      }
    }
    stage('Verify') {
      steps {
        sh '''
        sleep 5
        for s in $SERVERS; do
          ssh -o StrictHostKeyChecking=no ubuntu@$s "docker ps | grep learning-app && curl -sf http://localhost"
        done
        '''
      }
    }
  }
}

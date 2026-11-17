pipeline {
  agent any
  environment {
    APP    = 'sample-app'
    PORT   = '8081'      // poort op dmz010 (enkel 127.0.0.1) en op infra010 (staging)
    CPORT  = '5050'      // poort waarop de Flask-app in de container luistert
    IMAGE  = "${APP}:${BUILD_NUMBER}"
    DEPLOY = 'deploy@192.0.2.10'
  }
  stages {
    stage('Build') {
      steps { sh 'docker build -t $IMAGE .' }
    }
    stage('Deploy staging') {
      steps {
        sh '''
          docker rm -f $APP-staging || true
          docker run -d --name $APP-staging -p $PORT:$CPORT $IMAGE
        '''
      }
    }
    stage('Deploy production') {
      steps {
        sshagent(['dmz010-ssh']) {
          sh '''
            docker save $IMAGE | ssh -o StrictHostKeyChecking=no $DEPLOY docker load
            ssh -o StrictHostKeyChecking=no $DEPLOY \
              "docker rm -f $APP || true; docker run -d --name $APP --restart unless-stopped -p 127.0.0.1:$PORT:$CPORT $IMAGE"
          '''
        }
      }
    }
  }
}
pipeline {
  agent any

  options {
    timeout(time: 2, unit: 'MINUTES')
  }

  environment {
      //DOCKER_IMAGE_NAME = "192.168.100.20:5000/testapp"
      DOCKER_REGISTRY_SERVER = "192.168.100.20:5000"
      APP_NAME =  "testapp"
      DEPLOY_SERVER = "192.168.100.21"
      DEPLOY_USER = "root"
  }
  
  stages {

    stage('Building image') {
      steps{
          sh "sudo docker build -t ${env.APP_NAME} ."  
        }
    }    

    stage('Run tests') {
      steps {
        sh "sudo docker run ${env.APP_NAME} npm test"
      }
    }

    stage('Tag image') {
      steps {
        sh """
          sudo docker tag ${env.APP_NAME} ${env.DOCKER_REGISTRY_SERVER}/${env.APP_NAME}
        """
      }
    }

    stage('Push image to registry') {
      steps {
        sh """
          sudo docker push ${env.DOCKER_REGISTRY_SERVER}/${env.APP_NAME}
        """
      }
    }

    stage('Deploy') {
      when {
        branch 'dev'
      }

      steps {
        script {
          withCredentials([sshUserPrivateKey(credentialsId: 'deploy-ssh-key', keyFileVariable: 'SSH_KEY')]) {
            sh '''
              ssh -o StrictHostKeyChecking=no -i $SSH_KEY $DEPLOY_USER@$DEPLOY_SERVER bash -c "
                docker pull $DOCKER_REGISTRY_SERVER/$APP_NAME:$BRANCH_NAME-$BUILD_NUMBER &&
                docker stop $APP_NAME || true &&
                docker rm $APP_NAME || true &&
                docker run -d --name $APP_NAME -p 3000:3000 $DOCKER_REGISTRY_SERVER/$APP_NAME:$BRANCH_NAME-$BUILD_NUMBER
              "
            '''
          }
        }
      }
    }
  }
}
  


pipeline {
  agent any

  options {
    timeout(time: 2, unit: 'MINUTES')
  }

  environment {
      //DOCKER_IMAGE_NAME = "192.168.100.20:5000/testapp"
      DOCKER_REGISTRY_SERVER = "192.168.100.20:5000"
      APP_NAME =  "pin1"
      DEV_SERVER = "192.168.100.21"
      DEV_USER = "root"
      PROD_SERVER = "192.168.100.22"
      PROD_USER = "root"
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
          sudo docker tag ${env.APP_NAME} ${env.DOCKER_REGISTRY_SERVER}/${env.APP_NAME}:${env.BRANCH_NAME}-${env.BUILD_NUMBER} && \
          sudo docker tag ${env.APP_NAME} ${env.DOCKER_REGISTRY_SERVER}/${env.APP_NAME}:latest
        """
      }
    }

    stage('Push image to registry') {
      steps {
        sh """
          sudo docker push ${env.DOCKER_REGISTRY_SERVER}/${env.APP_NAME}:${env.BRANCH_NAME}-${env.BUILD_NUMBER}
          sudo docker push ${env.DOCKER_REGISTRY_SERVER}/${env.APP_NAME}:latest
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
            sh """
              ssh -o StrictHostKeyChecking=no -i ${SSH_KEY} ${DEV_USER}@${DEV_SERVER} bash -c "
                docker pull ${env.DOCKER_REGISTRY_SERVER}/${env.APP_NAME}:${env.BRANCH_NAME}-${env.BUILD_NUMBER} &&
                docker stop ${env.APP_NAME} || true &&
                docker rm ${env.APP_NAME} || true &&
                docker run -d --name ${env.APP_NAME} -p 3000:3000 ${env.DOCKER_REGISTRY_SERVER}/${env.APP_NAME}:${env.BRANCH_NAME}-${env.BUILD_NUMBER}
              "
            """
          }
        }
      }

      when {
        branch 'stable'
      }

      steps {
        script {
          withCredentials([sshUserPrivateKey(credentialsId: 'deploy-ssh-key', keyFileVariable: 'SSH_KEY')]) {
            sh """
              ssh -o StrictHostKeyChecking=no -i ${SSH_KEY} ${PROD_USER}@${PROD_SERVER} bash -c "
                docker pull ${env.DOCKER_REGISTRY_SERVER}/${env.APP_NAME}:latest &&
                docker stop ${env.APP_NAME} || true &&
                docker rm ${env.APP_NAME} || true &&
                docker run -d --name ${env.APP_NAME} -p 3000:3000 ${env.DOCKER_REGISTRY_SERVER}/${env.APP_NAME}:latest
              "
            """
          }
        }
      }
    }
  }
}

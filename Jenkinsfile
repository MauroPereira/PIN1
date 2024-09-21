pipeline {
  agent any

  options {
    timeout(time: 2, unit: 'MINUTES')
  }

  environment {
      DOCKER_IMAGE_NAME = "192.168.100.20:5000/testapp"
      DEPLOY_SERVER = "192.168.100.21"
      DEPLOY_USER = "root"
  }
  
  stages {

    stage('Building image') {
        steps{
            sh '''
            sudo docker build -t testapp .
              '''  
          }
      }    

    stage('Run tests') {
      steps {
        sh "sudo docker run testapp npm test"
      }
    }

    stage('Push image to registry') {
          steps {
              sh '''
              # Etiquetar la imagen con el nombre del registro local
              sudo docker tag testapp localhost:5000/testapp
              
              # Empujar la imagen al registro
              sudo docker push localhost:5000/testapp
              '''
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
                      ssh -o StrictHostKeyChecking=no -i ${SSH_KEY} ${DEPLOY_USER}@${DEPLOY_SERVER} bash -c '
                          docker pull ${DOCKER_IMAGE_NAME}:${env.BRANCH_NAME}-${env.BUILD_NUMBER}
                          docker stop pin1-nodejs || true
                          docker rm pin1-nodejs || true
                          docker run -d --name pin1-nodejs -p 3000:3000 ${DOCKER_IMAGE_NAME}:${env.BRANCH_NAME}-${env.BUILD_NUMBER}
                      '
                  """
              }
          }
      }
    }
  }
}


    
  


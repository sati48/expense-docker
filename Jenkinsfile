pipeline {
  agent any

  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/sati48/expense-docker.git'
      }
    }

    stage('Build & Deploy') {
      steps {
        sshagent(['expense_vm_ssh']) {
          sh '''
          ssh -o StrictHostKeyChecking=no ec2-user@98.81.128.66 "
            cd /home/ec2-user/expense-docker &&
            git pull &&
            docker compose down &&
            docker compose up -d --build
          "
          '''
        }
      }
    }
  }
}

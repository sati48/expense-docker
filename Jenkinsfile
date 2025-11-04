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
        sshagent(['ec2-deploy']) {
          sh '''
          ssh -o StrictHostKeyChecking=no jenkins@54.90.83.183 "
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

pipeline {
    agent any
    
    environment {
        target_vm = "ec2-user@54.90.83.183"
    }
    
    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/sati48/expense-docker.git'
            }
        }

        stage('Build Docker Images') {
            steps {
                sh 'docker-compose build'
            }
        }

        stage('Deploy to VM') {
            steps {
                sshagent(['expense-vm-ssh']) {
                    sh '''
                    ssh - o StrictHostKeyChecking = no $target_vm "
                            cd ~/expense-docker &&
                    docker - compose down &&
                        docker - compose up - d--build
                    "
                    '''
                }
            }
        }
    }
    
    post {
        success {
            echo "✅ Deployment successful! Expense app is live on the target VM."
        }
        failure {
            echo "❌ Deployment failed. Check console logs for details."
        }
    }
}
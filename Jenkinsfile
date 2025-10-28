pipeline {
    agent any

    environment {
        IMAGE = "jayasuriya27/portfolio:latest"
        CRED = "Dcokerhub"
        EC2_USER = "ubuntu"
        EC2_IP = "13.200.210.146"  // update if changed
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/Jayasuriya-S-S/Portfolio.git'
            }
        }

        stage('Build Node Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Docker Build & Push') {
            steps {
                withCredentials([string(credentialsId: CRED, variable: 'TOKEN')]) {
                    sh '''
                    docker build -t $IMAGE .
                    echo "$TOKEN" | docker login -u "jayasuriya27" --password-stdin
                    docker push $IMAGE
                    '''
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                sh '''
                ssh -o StrictHostKeyChecking=no $EC2_USER@$EC2_IP "
                docker pull $IMAGE &&
                docker stop portfolio || true &&
                docker rm portfolio || true &&
                docker run -d -p 80:3000 --name portfolio $IMAGE
                "
                '''
            }
        }
    }
}

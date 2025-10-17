pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'npm install'
            }
        }
        stage('Docker Build & Push') {
            steps {
                withCredentials([string(credentialsId: 'dockerhub-token', variable: 'TOKEN')]) {
                    sh '''
                    docker build -t jayasuriya/portfolio:latest .
                    echo "$TOKEN" | docker login -u jayasuriya --password-stdin
                    docker push jayasuriya/portfolio:latest
                    '''
                }
            }
        }
        stage('Deploy to EC2') {
            steps {
                sh 'ssh -o StrictHostKeyChecking=no ubuntu@<EC2-IP> "docker pull jayasuriya/portfolio:latest && docker stop portfolio || true && docker rm portfolio || true && docker run -d -p 80:3000 --name portfolio jayasuriya/portfolio:latest"'
            }
        }
    }
}

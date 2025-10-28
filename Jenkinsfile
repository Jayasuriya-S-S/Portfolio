pipeline {
    agent any

    environment {
        IMAGE = "jaiswathi1234/portfolio-app"
        CRED = "Dcokerhub"
        EC2_USER = "ubuntu"
        EC2_IP = "13.200.210.146"
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
                withCredentials([usernamePassword(credentialsId: CRED, usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh '''
                    docker build -t $IMAGE .
                    echo "$PASS" | docker login -u "$USER" --password-stdin
                    docker push $IMAGE
                    '''
                }
            }
        }

       stage('Deploy to EC2') {
       steps {
        withCredentials([sshUserPrivateKey(credentialsId: 'ec2-key', keyFileVariable: 'KEY_FILE', usernameVariable: 'SSH_USER')]) {
            sh '''
            ssh -i $KEY_FILE -o StrictHostKeyChecking=no $SSH_USER@13.200.210.146 "
                docker pull jaiswathi1234/portfolio-app &&
                docker stop portfolio || true &&
                docker rm portfolio || true &&
                docker run -d -p 80:3000 --name portfolio jaiswathi1234/portfolio-app
            "
            '''
          }
       }
    }
}

pipeline {
    agent any

    environment {
        PATH = "/usr/bin:$PATH" // Ensure npm is accessible
        BUILD_DIR = 'dist' // Output folder for build artifacts
        EC2_USER = "ec2-user"
        EC2_IP = "3.111.135.252" // Replace with your EC2 public IP
        REMOTE_DIR = "/usr/share/nginx/html"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/shyam9307/country-viewer.git'
            }
        }
        stage('Install Angular CLI and Dependencies') {
            steps {
                sh 'node -v'
                sh 'npm -v'
                sh 'npm install @angular/cli@15.1.2 --save-dev' // Install locally instead of globally
                sh 'npm install'
            }
        }
        stage('Build Angular App') {
            steps {
                sh 'npx ng build --configuration production --output-path=${BUILD_DIR}' // Use npx
                sh 'ls -l ${BUILD_DIR}/'
            }
        }
        stage('Deploy to EC2') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'EC2_SSH_KEY', keyFileVariable: 'SSH_KEY_PATH', usernameVariable: 'SSH_USER')]) {
                    script {
                        sh """
                        ssh -o StrictHostKeyChecking=no -i ${SSH_KEY_PATH} ${EC2_USER}@${EC2_IP} '
                            sudo rm -rf ${REMOTE_DIR}/* &&
                            sudo mkdir -p ${REMOTE_DIR} &&
                            sudo chown ${EC2_USER}:${EC2_USER} ${REMOTE_DIR} &&
                            exit
                        '
                        """
                        sh "scp -o StrictHostKeyChecking=no -i ${SSH_KEY_PATH} -r ${BUILD_DIR}/* ${EC2_USER}@${EC2_IP}:${REMOTE_DIR}/"
                        sh "ssh -o StrictHostKeyChecking=no -i ${SSH_KEY_PATH} ${EC2_USER}@${EC2_IP} 'sudo systemctl restart nginx'"
                    }
                }
            }
        }
    }
}

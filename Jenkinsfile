pipeline {
    agent any

    environment {
        // Define deployment details.
        // REMOTE_DIR is the directory on EC2 where the entire project will be deployed.
        EC2_USER   = "ec2-user"
        EC2_IP     = "3.111.135.252"          // Replace with your EC2 public IP
        REMOTE_DIR = "/usr/share/nginx/html/country-viewer"
    }

    stages {
        stage('Checkout Code') {
            steps {
                // Checkout the project from GitHub.
                git branch: 'master', url: 'https://github.com/grraghav120/country-viewer'
            }
        }
        stage('Install Dependencies') {
            steps {
                // Print Node and npm versions for logging.
                sh 'node -v'
                sh 'npm -v'
                // Clean previous node modules and install fresh dependencies.
                sh 'rm -rf node_modules package-lock.json || true'
                sh 'npm cache clean --force'
                sh 'npm install'
            }
        }
        stage('Optional: Run Tests') {
            steps {
                // Uncomment if your project has tests.
                // sh 'npm test'
                echo 'Skipping tests for now.'
            }
        }
        stage('Deploy to EC2 and Run ng serve') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'EC2_SSH_KEY', keyFileVariable: 'SSH_KEY_PATH')]) {
                    script {
                        // Clean the remote deployment directory, create it, and ensure proper ownership.
                        def cleanCmd = """
                            ssh -o StrictHostKeyChecking=no -i ${SSH_KEY_PATH} ${EC2_USER}@${EC2_IP} '
                                sudo rm -rf ${REMOTE_DIR}/* &&
                                sudo mkdir -p ${REMOTE_DIR} &&
                                sudo chown ${EC2_USER}:${EC2_USER} ${REMOTE_DIR} &&
                                exit
                            '
                        """
                        echo "Executing clean command: ${cleanCmd}"
                        sh cleanCmd
                    }
                    // Copy the entire project to the remote directory.
                    // (Ensure that your .git folder isn’t necessary on the server. If needed, adjust your SCP pattern.)
                    sh "scp -o StrictHostKeyChecking=no -i ${SSH_KEY_PATH} -r * ${EC2_USER}@${EC2_IP}:${REMOTE_DIR}/"

                    // On the remote host, change to the deployed directory,
                    // install dependencies (in case they’re not copied), and run 'ng serve' in the background.
                    def serveCmd = """
                        ssh -o StrictHostKeyChecking=no -i ${SSH_KEY_PATH} ${EC2_USER}@${EC2_IP} '
                            cd ${REMOTE_DIR} &&
                            npm install &&
                            nohup ng serve --host 0.0.0.0 --port 4200 > ngserve.log 2>&1 &
                            exit
                        '
                    """
                    echo "Executing serve command: ${serveCmd}"
                    sh serveCmd
                }
            }
        }
    }
}

pipeline {
    agent any

    environment {
        SERVER_IP = credentials('prod-server-ip')
    }
    stages {
        stage('Setup') {
            steps {
                script {
                    // Ensure the virtual environment is created before installing dependencies
                    sh 'python3 -m venv venv'
                    sh './venv/bin/pip install -r requirements.txt'
                }
            }
        }
        stage('Test') {
            steps {
                // Run tests in the virtual environment
                script {
                    sh './venv/bin/pytest'
                }
            }
        }

        stage('Package code') {
            steps {
                sh "zip -r myapp.zip ./* -x '*.git*'"
                sh "ls -lart"
            }
        }

        stage('Deploy to Prod') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'ssh-key', keyFileVariable: 'nov2024', usernameVariable: 'username')]) {
                    script {
                        // Ensure a virtual environment is created on the production server
                        sh '''
                        scp -i $nov2024 -o StrictHostKeyChecking=no myapp.zip  ${username}@${SERVER_IP}:/home/ubuntu/
                        ssh -i $nov2024 -o StrictHostKeyChecking=no ${username}@${SERVER_IP} << EOF
                            # Ensure directory exists and extract app
                            mkdir -p /home/ubuntu/app/
                            unzip -o /home/ubuntu/myapp.zip -d /home/ubuntu/app/
                            
                            # Create and activate virtual environment
                            cd /home/ubuntu/app/
                            python3 -m venv venv
                            source venv/bin/activate
                            
                            # Install requirements inside the virtual environment
                            pip install -r requirements.txt
                            
                            # Restart the service to apply changes
                            sudo systemctl restart flaskapp.service
EOF
                        '''
                    }
                }
            }
        }
    }
}

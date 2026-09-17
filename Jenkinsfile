pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

         stage('Get Git Commit ID') {
            steps {
                bat 'git rev-parse --short HEAD'
            }
        }

        stage('Build Docker Image') {
            steps {
                bat "docker build -t jenkins-react-app:%BUILD_NUMBER% ."
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'docker-hub-cred',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    bat '''
                        docker login -u %DOCKER_USERNAME% -p %DOCKER_PASSWORD%
                        docker tag jenkins-react-app:%BUILD_NUMBER% %DOCKER_USERNAME%/jenkins-react-app:%BUILD_NUMBER%
                        docker push %DOCKER_USERNAME%/jenkins-react-app:%BUILD_NUMBER%
                    '''
                }
            }
        }

        stage('Deploy to Oracle') {
                steps {
                    sshagent(credentials: ['oracle-vm-ssh']) {
                        bat '''
                            ssh -o StrictHostKeyChecking=no ubuntu@129.154.45.228 "sudo docker pull yashasvi2000/jenkins-react-app:%BUILD_NUMBER%"
                            ssh -o StrictHostKeyChecking=no ubuntu@129.154.45.228  "sudo docker rm -f jenkins-react-container || true"
                            ssh -o StrictHostKeyChecking=no ubuntu@129.154.45.228  "sudo docker run -d --name jenkins-react-container -p 8080:80 yashasvi2000/jenkins-react-app:%BUILD_NUMBER%"
                        '''
                    }
                }
            }
    }
}

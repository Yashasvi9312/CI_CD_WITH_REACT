pipeline {
    agent any

    parameters {
        string(
            name: 'ROLLBACK_SHA',
            defaultValue: '',
            description: 'Git SHA of the Docker image to rollback to'
        )
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Get Git Commit ID') {
            steps {
                script {
                    env.GIT_COMMIT_SHORT = bat(
                        script: '@git rev-parse --short HEAD',
                        returnStdout: true
                    ).trim()

                    echo "Git Commit: ${env.GIT_COMMIT_SHORT}"
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                bat "docker build -t jenkins-react-app:${env.GIT_COMMIT_SHORT} ."
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

                        docker tag jenkins-react-app:%GIT_COMMIT_SHORT% %DOCKER_USERNAME%/jenkins-react-app:%GIT_COMMIT_SHORT%

                        docker push %DOCKER_USERNAME%/jenkins-react-app:%GIT_COMMIT_SHORT%
                    '''
                }
            }
        }

        stage('Rollback') {
                when {
                    expression {
                        return params.ROLLBACK_SHA?.trim()
                    }
                }

                steps {
                    sshagent(credentials: ['oracle-vm-ssh']) {
                        bat '''
                            ssh -o StrictHostKeyChecking=no ubuntu@129.154.45.228 "sudo docker pull yashasvi2000/jenkins-react-app:%ROLLBACK_SHA%"

                            ssh -o StrictHostKeyChecking=no ubuntu@129.154.45.228 "sudo docker rm -f jenkins-react-container || true"

                            ssh -o StrictHostKeyChecking=no ubuntu@129.154.45.228 "sudo docker run -d --name jenkins-react-container -p 8080:80 yashasvi2000/jenkins-react-app:%ROLLBACK_SHA%"

                            ssh -o StrictHostKeyChecking=no ubuntu@129.154.45.228 "curl -f -s http://localhost:8080 > /dev/null"

                            ssh -o StrictHostKeyChecking=no ubuntu@129.154.45.228 "sudo docker ps --filter name=jenkins-react-container"
                        '''
                    }
                }
            }
    }
}

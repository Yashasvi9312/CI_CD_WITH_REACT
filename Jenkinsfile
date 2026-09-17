def saveLastKnownGood(String imageTag) {
    bat """
        if not exist C:\\jenkins-lkg mkdir C:\\jenkins-lkg
        echo ${imageTag} > C:\\jenkins-lkg\\last-known-good.txt
    """
}

def getLastKnownGood() {
    def output = bat(
        script: '@type C:\\jenkins-lkg\\last-known-good.txt',
        returnStdout: true
    ).trim()

    return output
}
def deployToOracle(String imageTag) {
    sshagent(credentials: ['oracle-vm-ssh']) {
        bat """
            ssh -o StrictHostKeyChecking=no ubuntu@129.154.45.228 "sudo docker pull yashasvi2000/jenkins-react-app:${imageTag}"

            ssh -o StrictHostKeyChecking=no ubuntu@129.154.45.228 "sudo docker rm -f jenkins-react-container || true"

            ssh -o StrictHostKeyChecking=no ubuntu@129.154.45.228 "sudo docker run -d --name jenkins-react-container -p 8080:80 yashasvi2000/jenkins-react-app:${imageTag}"

            ssh -o StrictHostKeyChecking=no ubuntu@129.154.45.228 "curl -f -s http://localhost:8080 > /dev/null"

            ssh -o StrictHostKeyChecking=no ubuntu@129.154.45.228 "sudo docker ps --filter name=jenkins-react-container"
        """
    }
}

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
            when {
                expression {
                    return !params.ROLLBACK_SHA?.trim()
                }
            }

            steps {
                bat "docker build -t jenkins-react-app:${env.GIT_COMMIT_SHORT} ."
            }
        }

        stage('Push to Docker Hub') {
            when {
                expression {
                    return !params.ROLLBACK_SHA?.trim()
                }
            }

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

        stage('Deploy to Oracle') {
                when {
                    expression {
                        return !params.ROLLBACK_SHA?.trim()
                    }
                }

                steps {
                    script {

                        try {
                            deployToOracle(env.GIT_COMMIT_SHORT)

                            saveLastKnownGood(env.GIT_COMMIT_SHORT)

                            echo "Deployment successful."
                            echo "Last Known Good: ${env.GIT_COMMIT_SHORT}"

                        } catch (Exception e) {

                            echo "Deployment failed!"
                            echo "Starting automatic rollback..."

                            def lastKnownGood = getLastKnownGood()

                            if (!lastKnownGood) {
                                error "No Last Known Good version available for rollback."
                            }

                            echo "Rolling back to: ${lastKnownGood}"

                            deployToOracle(lastKnownGood)

                            echo "Rollback successful: ${lastKnownGood}"

                            throw e
                        }
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
                 script {
                    deployToOracle(params.ROLLBACK_SHA)
                }
            }
        }
    }

    post{
    success{
        script{
            def text = getLastKnownGood()
            echo "${text}"
        }
    }
}
}
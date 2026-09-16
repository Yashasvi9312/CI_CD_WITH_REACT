pipeline{
    agent any 

    tools{
        nodejs 'node24'
    }

    environment{
        SECRET_TEXT = credentials('test-secret')
    }

    stages{
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install') {
            steps {
                bat 'npm ci'
            }
        }

        stage('Lint'){
            steps{
                bat 'npm run lint'
            }
        }

        stage('Test'){
            steps{
                bat 'npm run test'
            }
        }

        stage('Build') {
            steps {
                bat 'npm run build'
            }
        }

        stage('Test Credentials') {
            steps {
                echo "Secret is available to Jenkins"
            }
        }

        stage('Test Username Password') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'demo-user-pass',
                        usernameVariable: 'DEMO_USER',
                        passwordVariable: 'DEMO_PASSWORD'
                    )
                ]) {
                    bat 'echo Username is %DEMO_USER%'
                    bat 'echo Password is %DEMO_PASSWORD%'
                }
            }
        }

        stage('Archive Artifacts'){
            steps{
                archiveArtifacts artifacts: 'dist/**',fingerprint: true
            }
        }

    }

    post{
        success{
            echo 'React CI pipeline completed successfully!'

            build job : 'jenkins-react-cd',
            parameters:[
                string(
                    name:'BUILD_NUMBER_TO_DEPLOY',
                    value:env.BUILD_NUMBER
                )
            ]
        }
        failure{
            echo 'React CI pipeline failed!'
        }
    }
}

pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
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

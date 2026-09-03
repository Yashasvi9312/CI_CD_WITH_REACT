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
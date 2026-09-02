pipeline{
    agent any 

    tools{
        nodejs 'node24'
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

        stage('Build') {
            steps {
                bat 'npm run build'
            }
        }
    }

    post{
        success{
            echo 'React CI pipeline completed successfully!'
        }
        failure{
            echo 'React CI pipeline failed!'
        }
    }
}
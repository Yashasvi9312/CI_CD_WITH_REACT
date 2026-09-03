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

        stage('Archive Artifacts'){
            steps{
                archiveArtifacts artifacts: 'dist/**',fingerprint: true
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
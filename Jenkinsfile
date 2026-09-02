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

        stage('Deploy') {
            steps {
                bat 'if not exist C:\\jenkins-deployed-app mkdir C:\\jenkins-deployed-app'
                bat 'xcopy /E /I /Y dist C:\\jenkins-deployed-app'
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
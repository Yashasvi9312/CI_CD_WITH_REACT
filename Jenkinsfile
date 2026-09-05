// pipeline{
//     agent any 

//     tools{
//         nodejs 'node24'
//     }

//     environment{
//         SECRET_TEXT = credentials('test-secret')
//     }

//     stages{
//         stage('Checkout') {
//             steps {
//                 checkout scm
//             }
//         }

//         stage('Install') {
//             steps {
//                 bat 'npm ci'
//             }
//         }

//         stage('Lint'){
//             steps{
//                 bat 'npm run lint'
//             }
//         }

//         stage('Test'){
//             steps{
//                 bat 'npm run test'
//             }
//         }

//         stage('Build') {
//             steps {
//                 bat 'npm run build'
//             }
//         }

//         stage('Test Credentials') {
//             steps {
//                 echo "Secret is available to Jenkins"
//             }
//         }

//         stage('Test Username Password') {
//             steps {
//                 withCredentials([
//                     usernamePassword(
//                         credentialsId: 'demo-user-pass',
//                         usernameVariable: 'DEMO_USER',
//                         passwordVariable: 'DEMO_PASSWORD'
//                     )
//                 ]) {
//                     bat 'echo Username is %DEMO_USER%'
//                     bat 'echo Password is %DEMO_PASSWORD%'
//                 }
//             }
//         }

//         stage('Archive Artifacts'){
//             steps{
//                 archiveArtifacts artifacts: 'dist/**',fingerprint: true
//             }
//         }

//     }

//     post{
//         success{
//             echo 'React CI pipeline completed successfully!'

//             build job : 'jenkins-react-cd',
//             parameters:[
//                 string(
//                     name:'BUILD_NUMBER_TO_DEPLOY',
//                     value:env.BUILD_NUMBER
//                 )
//             ]
//         }
//         failure{
//             echo 'React CI pipeline failed!'
//         }
//     }
// }

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
                bat 'docker build -t jenkins-react-app:1.0 .'
            }
        }

        stage('Remove Old Container') {
            steps {
                bat 'docker rm -f jenkins-react-container || exit 0'
            }
        }

        stage('Run New Container') {
            steps {
                bat 'docker run -d --name jenkins-react-container -p 3000:80 jenkins-react-app:1.0'
            }
        }
    }
}
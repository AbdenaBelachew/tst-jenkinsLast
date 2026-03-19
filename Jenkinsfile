pipeline {
    agent any

    tools {
        nodejs 'NodeJS-18'
    }

    stages {

        stage('Checkout Code') {
            steps {
                git 'https://github.com/AbdenaBelachew/tst-jenkinsLast.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                bat 'npm install'
                // use 'sh' if Linux
            }
        }

        stage('Build Project') {
            steps {
                bat 'npm run build'
            }
        }

        stage('Archive Build') {
            steps {
                archiveArtifacts artifacts: 'build/**', fingerprint: true
            }
        }

    }
}

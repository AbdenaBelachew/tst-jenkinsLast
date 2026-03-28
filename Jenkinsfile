pipeline {
    agent any

    tools {
        nodejs 'node18'
    }

    stages {

        stage('Install Dependencies') {
            steps {
                bat 'npm install'
            }
        }

        stage('Build Project') {
            steps {
                bat 'npm run build'
            }
        }

        stage('Archive Build') {
            steps {
                archiveArtifacts artifacts: 'dist/**', fingerprint: true
            }
        }

        stage('Deploy to IIS') {
            steps {
                powershell '''
                $source = "${env:WORKSPACE}\\dist"
                $destination = "C:\\inetpub\\wwwroot\\myapp"
                if (!(Test-Path $destination)) {
                    New-Item -Path $destination -ItemType Directory -Force
                }
                Copy-Item -Path $source\\* -Destination $destination -Recurse -Force
                Write-Host "Deployment complete!"
                '''
            }
        }

    }

    post {
        success {
            echo 'CI/CD pipeline finished successfully!'
        }
        failure {
            echo 'Pipeline failed. Check logs.'
        }
    }
}

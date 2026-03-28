pipeline {
    agent any

    tools {
        nodejs 'node18'
    }

    stages {

        stage('Install Frontend Dependencies') {
            steps {
                dir('frontend') {
                    bat 'npm install'
                }
            }
        }

        stage('Build Frontend') {
            steps {
                dir('frontend') {
                    bat 'npm run build'
                }
            }
        }

        stage('Install Backend Dependencies') {
            steps {
                dir('backend') {
                    bat 'npm install'
                }
            }
        }

        stage('Archive Build') {
            steps {
                archiveArtifacts artifacts: 'frontend/dist/**', fingerprint: true
            }
        }

        stage('Deploy Frontend') {
            steps {
                powershell '''
                $source = "${env:WORKSPACE}\\frontend\\dist"
                $destination = "C:\\inetpub\\wwwroot\\myapp"
                if (!(Test-Path $destination)) {
                    New-Item -Path $destination -ItemType Directory -Force
                }
                Copy-Item -Path $source\\* -Destination $destination -Recurse -Force
                Write-Host "Frontend deployed!"

                # Manage IIS App
                Import-Module WebAdministration
                $siteName = "Default Web Site"
                $appPath = "myapp"
                if (-Not (Test-Path "IIS:\\Sites\\$siteName\\$appPath")) {
                    New-WebApplication -Name $appPath -Site $siteName -PhysicalPath $destination -ApplicationPool "DefaultAppPool"
                } else {
                    Set-ItemProperty "IIS:\\Sites\\$siteName\\$appPath" -Name physicalPath -Value $destination
                }
                '''
            }
        }

        stage('Deploy Backend') {
            steps {
                powershell '''
                $source = "${env:WORKSPACE}\\backend"
                $destination = "C:\\inetpub\\backend\\mybackend"

                # Stop existing node processes (WARNING: stops all node apps)
                Stop-Process -Name node -ErrorAction SilentlyContinue

                if (!(Test-Path $destination)) {
                    New-Item -Path $destination -ItemType Directory -Force
                }
                Copy-Item -Path $source\\* -Destination $destination -Recurse -Force

                # Start backend in background
                Start-Process "node" -ArgumentList "$destination\\index.js" -WorkingDirectory $destination
                Write-Host "Backend deployed and started!"
                '''
            }
        }

    }

    post {
        success {
            echo 'Full Stack CI/CD pipeline finished successfully!'
        }
        failure {
            echo 'Pipeline failed. Check logs.'
        }
    }
}

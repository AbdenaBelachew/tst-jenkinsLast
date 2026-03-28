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
                # Clean up old deployment and web.config (to avoid 500 errors)
                if (Test-Path "$destination\\web.config") {
                    Remove-Item "$destination\\web.config" -Force
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
                $port = 3001

                # Clean up existing processes on port 3001
                $processId = (Get-NetTCPConnection -LocalPort $port -ErrorAction SilentlyContinue).OwningProcess
                if ($processId) {
                    Write-Host "Killing process on port $port (PID: $processId)..."
                    Stop-Process -Id $processId -Force -ErrorAction SilentlyContinue
                }

                if (!(Test-Path $destination)) {
                    New-Item -Path $destination -ItemType Directory -Force
                }
                Copy-Item -Path $source\\* -Destination $destination -Recurse -Force

                # Prevent Jenkins from killing the process
                $env:BUILD_ID = "dontKillMe"

                # Start backend and redirect logs to a file for debugging
                Write-Host "Starting backend..."
                Start-Process "node" -ArgumentList "index.js" -WorkingDirectory $destination -RedirectStandardOutput "$destination\\backend.log" -RedirectStandardError "$destination\\backend_error.log"
                
                # Verify if port 3001 is now listening
                Start-Sleep -Seconds 5
                if (Get-NetTCPConnection -LocalPort $port -ErrorAction SilentlyContinue) {
                    Write-Host "SUCCESS: Backend is listening on port $port!"
                } else {
                    Write-Host "ERROR: Backend failed to bind to port $port. Check $destination\\backend_error.log for details."
                    # Print the error log for visibility if it exists
                    if (Test-Path "$destination\\backend_error.log") {
                        Get-Content "$destination\\backend_error.log"
                    }
                    exit 1
                }
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

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

                # Ensure destination folder exists
                if (!(Test-Path $destination)) {
                    New-Item -Path $destination -ItemType Directory -Force
                }

                # Copy files
                Copy-Item -Path $source\\* -Destination $destination -Recurse -Force
                Write-Host "Files deployed to IIS folder!"

                # Convert to IIS Application
                Import-Module WebAdministration
                $siteName = "Default Web Site"
                $appPath = "myapp"
                $appPoolName = "DefaultAppPool"

                if (-Not (Test-Path "IIS:\\Sites\\$siteName\\$appPath")) {
                    New-WebApplication -Name $appPath -Site $siteName -PhysicalPath $destination -ApplicationPool $appPoolName
                    Write-Host "IIS Application '$appPath' created successfully!"
                } else {
                    Set-ItemProperty "IIS:\\Sites\\$siteName\\$appPath" -Name physicalPath -Value $destination
                    Write-Host "IIS Application '$appPath' updated successfully!"
                }
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

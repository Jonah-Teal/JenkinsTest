
pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
        timestamps()
        disableConcurrentBuilds()
    }

    environment {
        IMAGE_NAME = 'JAK-TestContainer'
        DOCKERHUB_USERNAME = 'jonahteal'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                echo "Building branch: ${env.BRANCH_NAME}"
            }
        }

        stage('Validate Website') {
            steps {
                powershell '''
                    $ErrorActionPreference = "Stop"

                    if (-not (Test-Path "index.html")) {
                        throw "index.html is missing."
                    }

                    if (-not (Test-Path "Dockerfile")) {
                        throw "Dockerfile is missing."
                    }

                    Write-Host "Website files validated successfully."
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                powershell '''
                    $ErrorActionPreference = "Stop"

                    docker version
                    if ($LASTEXITCODE -ne 0) {
                        throw "Docker is unavailable."
                    }

                    docker build `
                        -t "$env:IMAGE_NAME`:$env:BUILD_NUMBER" .

                    if ($LASTEXITCODE -ne 0) {
                        throw "Docker build failed."
                    }

                    Write-Host "Docker image built successfully."
                '''
            }
        }

        stage('Test Website Container') {
            steps {
                powershell '''
                    $ErrorActionPreference = "Stop"

                    $container = "jenkins-test-$env:BUILD_NUMBER"

                    try {
                        docker run -d `
                            --name $container `
                            -p 18080:80 `
                            "$env:IMAGE_NAME`:$env:BUILD_NUMBER"

                        if ($LASTEXITCODE -ne 0) {
                            throw "Container failed to start."
                        }

                        $healthy = $false

                        for ($i = 0; $i -lt 15; $i++) {
                            Start-Sleep -Seconds 2

                            try {
                                $response = Invoke-WebRequest `
                                    -Uri "http://localhost:18080" `
                                    -TimeoutSec 3

                                if ($response.StatusCode -eq 200) {
                                    $healthy = $true
                                    break
                                }
                            } catch {
                                # Retry while the web server starts.
                            }
                        }

                        if (-not $healthy) {
                            docker logs $container
                            throw "Website HTTP health check failed."
                        }

                        Write-Host "Website returned HTTP 200."
                    }
                    finally {
                        docker rm -f $container 2>$null
                    }
                '''
            }
        }

        stage('Push to Docker Hub') {
            when {
                branch 'master'
            }

            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    powershell '''
                        $ErrorActionPreference = "Stop"

                        $tag = "$env:DOCKER_USER/$env:IMAGE_NAME`:$env:BUILD_NUMBER"

                        try {
                            $env:DOCKER_PASSWORD |
                                docker login `
                                    --username $env:DOCKER_USER `
                                    --password-stdin

                            if ($LASTEXITCODE -ne 0) {
                                throw "Docker Hub login failed."
                            }

                            docker tag `
                                "$env:IMAGE_NAME`:$env:BUILD_NUMBER" $tag

                            if ($LASTEXITCODE -ne 0) {
                                throw "Docker tag failed."
                            }

                            docker push $tag

                            if ($LASTEXITCODE -ne 0) {
                                throw "Docker push failed."
                            }

                            Write-Host "Image pushed to Docker Hub: $tag"
                        }
                        finally {
                            docker logout
                        }
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }

        failure {
            echo 'Pipeline failed. Check the console output.'
        }

        always {
            echo 'Pipeline execution finished.'
        }
    }
}

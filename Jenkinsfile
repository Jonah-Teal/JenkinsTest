
pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
        timestamps()
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                echo 'Source code checked out successfully.'
            }
        }

        stage('Build') {
            steps {
                echo "Building branch: ${env.BRANCH_NAME}"
                echo 'Build stage completed.'
            }
        }

        stage('Test') {
            steps {
                echo 'Running tests...'
                echo 'Test stage completed.'
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

stage('Test Docker Login') {
    steps {
        withCredentials([usernamePassword(
            credentialsId: '9766f83d-7c0b-4bde-95f4-103ec74e1d22',
            usernameVariable: 'jonahteal',
            passwordVariable: 'DOCKER_PASS'
        )]) {
            bat "echo %DOCKER_PASS% | docker login -u %DOCKER_USER% --password-stdin"
        }
    }
}

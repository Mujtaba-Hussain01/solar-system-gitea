pipeline {
    agent any

    stages {
        stage('Vm node Version') {
            steps {
                sh '''
                    node -v
                    npm -v
                '''
            }
        }
    }
}
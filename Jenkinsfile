pipeline {
    agent any

    tools {
        nodejs 'node-24.9.0'
    }

    stages {
        stage('VM Node Version') {
            steps {
                sh '''
                    node -v
                    npm -v
                '''
            }
        }
    }
}
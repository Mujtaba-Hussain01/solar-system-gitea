pipeline {
    agent any

    tools {
        nodejs 'Nodejs-20-19-5'
    }

    stages {
        stage('Install Dependencies') {
            steps {
                sh 'npm install --no-audit'
            }
        }

        stage('Dependencies Scanning') {
            parallel{
                stage('NPM Dependencies Audit') {
                    steps {
                        sh '''
                            npm audit --audit-level=critical
                            echo $?
                        '''
                    }
                }

                stage('OWASP Dependency Check') {
                    steps {
                        DependencyCheck additionalArguments: '''
                            --scan . 
                            --format 'ALL'
                            --out ./dependency-check-report
                            --pretty-print''', odcInstallation: 'OWASP-DepCheck-10'
                        
                    }
                }
            }
        }
    }
}
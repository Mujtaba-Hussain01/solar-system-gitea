pipeline {
    agent any

    tools {
        nodejs 'Nodejs-20-19-5'
    }

    environment {
        MONGO_URI = ""mongodb://mujtaba:mujtaba@1234@localhost:27017/superData""
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
                        dependencyCheck additionalArguments: '''
                            --scan ./ 
                            --out ./
                            --format ALL
                            --prettyPrint
                            --noupdate''', odcInstallation: 'OWASP-DepCheck-10'

                        dependencyCheckPublisher failedTotalCritical: 1, pattern: 'dependency-check-report.xml', stopBuild: true

                        publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, icon: '', keepAll: true, reportDir: './', reportFiles: 'dependency-check-jenkins.html', reportName: 'dependency check HTML Report', reportTitles: 'HTML Report', useWrapperFileDirectly: true])

                        junit allowEmptyResults: true, testResults: 'dependency-check-junit.xml'


                    }
                }
            }
        }

        stage('Unit Testing') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'mongo_db_creds', passwordVariable: 'MONGO_PASSWORD', usernameVariable: 'MONGO_USERNAME')])
                    sh 'npm test'
            
                junit allowEmptyResults: true, testResults: 'test_results.xml'
            }
        }
    }
}
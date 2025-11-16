pipeline {
    agent any

    tools {
        nodejs 'Nodejs-20-19-5'
    }

    environment {
        MONGO_USERNAME = credentials('mongodb_username')
        MONGO_PASSWORD = credentials('mongodb_password')
        MONGO_URI = "mongodb+srv://${MONGO_USERNAME}:${MONGO_PASSWORD}@us-visa.xixp6rv.mongodb.net/superData"
    }

    stages {
        stage('Install Dependencies') {
            options {timestamps()}
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
                    options {retry(2)}
                    steps {
                        dependencyCheck additionalArguments: '''
                            --scan ./ 
                            --out ./
                            --format ALL
                            --prettyPrint
                            --noupdate''', odcInstallation: 'OWASP-DepCheck-10'

                        dependencyCheckPublisher failedTotalCritical: 1, pattern: 'dependency-check-report.xml', stopBuild: true

                    }
                }
            }
        }

        stage('Unit Testing') {
            steps {

                sh 'echo colon-separated-credentials: $MONGO_DB_CREDS - $MONGO_USERNAME : $MONGO_PASSWORD'
                
                sh 'npm test'
            }
        }

        stage('Code Coverage') {
            steps {
                catchError(buildResult: 'SUCCESS', message: 'oops! will fix in future', stageResult: 'UNSTABLE') {
                    sh 'npm run coverage'
                }
            }
        }
    }

    post {
        always {
            junit allowEmptyResults: true, testResults: 'dependency-check-junit.xml'
            junit allowEmptyResults: true, testResults: 'test_results.xml'

            publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, icon: '', keepAll: true, reportDir: './', reportFiles: 'dependency-check-jenkins.html', reportName: 'dependency check HTML Report', reportTitles: 'HTML Report', useWrapperFileDirectly: true])
            publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, icon: '', keepAll: true, reportDir: 'coverage/lcov-report', reportFiles: 'index.html', reportName: 'Code Coverage HTML Report', reportTitles: '', useWrapperFileDirectly: true])
        }
    }
}
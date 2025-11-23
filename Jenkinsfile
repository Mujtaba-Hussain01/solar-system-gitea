pipeline {
    agent any

    tools {
        nodejs 'Nodejs-20-19-5'
        
    }

    environment {
        MONGO_USERNAME = credentials('mongodb_username')
        MONGO_PASSWORD = credentials('mongodb_password')
        MONGO_URI = "mongodb+srv://${MONGO_USERNAME}:${MONGO_PASSWORD}@us-visa.xixp6rv.mongodb.net/superData"
        // SONAR_SCANNER_HOME = tool 'SonarQube-Scanner-610'
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
                            --disableYarnAudit \
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

        stage('SAST - sonarQube Analysis') {
            steps {
                timeout(time: 60, unit: 'SECONDS') {
                    withSonarQubeEnv('sonar-qube-server'){
                        sh '''
                            sonar \
                                -Dsonar.sources=app.js \
                                -Dsonar.projectKey=sonarqube \
                                -Dsonar.javascript.lcov.reportPaths=./coverage/lcov.info
                        '''
                    }

                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'printenv'
                sh 'docker build -t mujtaba7794/solar-system-gitea:latest .'
                
            }
        }

        stage('Trivy Vulnerability Scanner') {
            steps {
                sh '''
                    echo "🔍 Running Trivy scan for LOW and MEDIUM vulnerabilities..."
                    trivy image mujtaba7794/solar-system-gitea:latest \
                        --severity LOW,MEDIUM \
                        --exit-code 0 \
                        --quiet \
                        --format json -o trivy-image-MEDIUM-results.json

                    echo "🔍 Running Trivy scan for HIGH and CRITICAL vulnerabilities..."
                    trivy image mujtaba7794/solar-system-gitea:latest \
                        --severity HIGH,CRITICAL \
                        --exit-code 1 \
                        --quiet \
                        --format json -o trivy-image-CRITICAL-results.json
                '''
            }

            post {
                always {
                    sh '''
                        echo "📄 Converting MEDIUM JSON → HTML..."
                        trivy convert \
                            --format template --template "@/usr/local/share/trivy/templates/html.tpl" \
                            --output trivy-image-MEDIUM-results.html trivy-image-MEDIUM-results.json

                        echo "📄 Converting CRITICAL JSON → HTML..."
                        trivy convert \
                            --format template --template "@/usr/local/share/trivy/templates/html.tpl" \
                            --output trivy-image-CRITICAL-results.html trivy-image-CRITICAL-results.json

                        echo "📄 Converting MEDIUM JSON → JUnit XML..."
                        trivy convert \
                            --format template --template "@/usr/local/share/trivy/templates/junit.tpl" \
                            --output trivy-image-MEDIUM-results.xml trivy-image-MEDIUM-results.json

                        echo "📄 Converting CRITICAL JSON → JUnit XML..."
                        trivy convert \
                            --format template --template "@/usr/local/share/trivy/templates/junit.tpl" \
                            --output trivy-image-CRITICAL-results.xml trivy-image-CRITICAL-results.json
                    '''
                }
            }

        }

    }

    post {
        always {
            junit allowEmptyResults: true, testResults: 'dependency-check-junit.xml'
            junit allowEmptyResults: true, testResults: 'test_results.xml'
            junit allowEmptyResults: true, testResults: 'trivy-image-CRITICAL-results.xml'
            junit allowEmptyResults: true, testResults: 'trivy-image-MEDIUM-results.xml'
            
            publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, icon: '', keepAll: true, reportDir: './', reportFiles: 'trivy-image-CRITICAL-results.html', reportName: 'Trivy Image Critical Vul Report', reportTitles: 'HTML Report', useWrapperFileDirectly: true])
            publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, icon: '', keepAll: true, reportDir: './', reportFiles: 'trivy-image-MEDIUM-results.html', reportName: 'Trivy Image Medium Vul Report', reportTitles: 'HTML Report', useWrapperFileDirectly: true])

            publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, icon: '', keepAll: true, reportDir: './', reportFiles: 'dependency-check-jenkins.html', reportName: 'dependency check HTML Report', reportTitles: 'HTML Report', useWrapperFileDirectly: true])
            publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, icon: '', keepAll: true, reportDir: 'coverage/lcov-report', reportFiles: 'index.html', reportName: 'Code Coverage HTML Report', reportTitles: '', useWrapperFileDirectly: true])
        }
    }
}
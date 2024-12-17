pipeline {
    agent any

    tools {
        nodejs 'nodejs-22-6-0'
    }

    environment {
        MONGO_URI = "mongodb+srv://supercluster.d83jj.mongodb.net/superData"
        MONGO_DB_CREDS = credentials('mongo-db-credentials') // This stores username & password together
        MONGO_USERNAME = credentials('mongo-db-username')
        MONGO_PASSWORD = credentials('mongo-db-password')
        SONAR_SCANNER_HOME = tool 'sonarqube-scanner-610';
    }

    stages {
        stage('Installing Dependencies') {
            options { timestamps() }
            steps {
                sh 'npm install --no-audit'
            }
        }

        stage('Dependency Scanning') {
            parallel {
                stage('NPM Dependency Audit') {
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
                            --disableYarnAudit 
                            --prettyPrint
                        ''', odcInstallation: 'OWASP-DepCheck-10'

                        dependencyCheckPublisher failedTotalCritical: 1, pattern: 'dependency-check-report.xml', stopBuild: false
                    }
                }
            }
        }

        stage('Unit Testing') {
            options { retry(2) }
            steps {
                // Correct reference to Jenkins credentials
                sh 'echo Colon-Separated - $MONGO_DB_CREDS'
                sh 'echo Username - ${MONGO_USERNAME}'
                sh 'echo Password - ${MONGO_PASSWORD}'

                sh 'npm test'
            }
        }

        stage('Code Coverage') {
            steps {
                catchError(buildResult: 'SUCCESS', message: 'Oops! it will be fixed in future releases', stageResult: 'UNSTABLE') {
                    sh 'npm run coverage'
                }
            }
        }



        stage('SAST - SonarQube') {
            steps {
                // sh 'sleep 5s'
                    withSonarQubeEnv('sonar-qube-server') {
                        sh 'echo $SONAR_SCANNER_HOME'
                        sh '''
                            $SONAR_SCANNER_HOME/bin/sonar-scanner \
                                -Dsonar.projectKey=amit-webdev \
                                -Dsonar.sources=app.js \
                                -Dsonar.javascript.lcov.reportPaths=./coverage/lcov.info \
                                -Dsonar.host.url=https://sonarcloud.io
                        '''
                    }
                
            }
        }
    }

    post {
        always {
            // Publish test and dependency scan reports
            junit allowEmptyResults: true, stdioRetention: '', testResults: 'test-results.xml'
            junit allowEmptyResults: true, stdioRetention: '', testResults: 'dependency-check-junit.xml' 

            // Publish OWASP ZAP Report
            publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, keepAll: true, 
                reportDir: './', reportFiles: 'zap_report.html', 
                reportName: 'DAST - OWASP ZAP Report', useWrapperFileDirectly: true])

            // Publish Dependency Check Report
            publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, keepAll: true, 
                reportDir: './', reportFiles: 'dependency-check-jenkins.html', 
                reportName: 'Dependency Check HTML Report', useWrapperFileDirectly: true])

            // Publish Code Coverage Report
            publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, keepAll: true, 
                reportDir: 'coverage/lcov-report', reportFiles: 'index.html', 
                reportName: 'Code Coverage HTML Report', useWrapperFileDirectly: true])
        }
    }
}



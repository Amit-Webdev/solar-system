pipeline {
    agent any

    tools {
        nodejs 'nodejs-22-6-0'
    }

    environment {
      MONGO_URI = "mongodb+srv://supercluster.d83jj.mongodb.net/superData"
    }



    stages {
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
                            --scan './' 
                            --out './'  
                            --format 'ALL' 
                            --disableYarnAudit \
                            --prettyPrint''', odcInstallation: 'OWASP-DepCheck-10'

                        dependencyCheckPublisher failedTotalCritical: 1, pattern: 'dependency-check-report.xml', stopBuild: false
                    }
                }
            }
        }

        stage('Unit Testing') {
            steps{
                withCredentials([usernamePassword(credentialsId: 'mongo-db-credentials', passwordVariable: 'MONGO_PASSWORD', usernameVariable: 'MONGO_USERNAME')]) {
                  sh 'npm test -- --no-deprecation'
               }

               junit allowEmptyResults: true, stdioRetention: '', testResults: 'test-results.xml'
            }
        }
    }
}



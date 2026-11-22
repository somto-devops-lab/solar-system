pipeline {
    agent any
    triggers {
    githubPush()
    }
    tools {
    nodejs 'nodejs-v22.22.1'
    }
    environment {
    MONGO_URI = "mongodb+srv://cluster0.tpmafor.mongodb.net/superData?authSource=admin"
    }
    stages {
        stage('Insatlling Dependencies') {
            options {
                timestamps()
            }

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

            stage('Verify Dependency Versions') {
                steps {
                    sh '''
                        npm ls y18n
                        npm ls y18n --package-lock-only --all
                    '''
                }
            }

                stage('OWASP Dependency Check') {
                    steps {
                        dependencyCheck additionalArguments: '''
                            --scan './'
                            --out './'
                            --format 'ALL'
                            --prettyPrint
                            --disableYarnAudit
                            --suppression './dependency-check-suppressions.xml'
                        ''', odcInstallation: 'OWASP-DepCheck-10',
                        nvdCredentialsId: 'nvd-api-key'
                        dependencyCheckPublisher failedTotalCritical: 1, pattern: 'dependency-check-report.xml', stopBuild: true
                        junit allowEmptyResults: true, keepProperties: true, testResults: 'dependency-check-junit.xml'
                        publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, icon: '', keepAll: true, reportDir: './', reportFiles: 'dependency-check-report.html', reportName: 'Dependency HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                    }
                }
         
            } 
        }
        stage('Unit Testing') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'Mongodb_Creden',
                    passwordVariable: 'MONGO_PASSWORD',
                    usernameVariable: 'MONGO_USERNAME'
                )]) {
                    sh 'npm test'
                }

                junit allowEmptyResults: true,
                    keepProperties: true,
                    testResults: 'test-results.xml'
            }
        }
        stage('Code Coverage') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'Mongodb_Creden',
                    passwordVariable: 'MONGO_PASSWORD',
                    usernameVariable: 'MONGO_USERNAME'
                )]) {
                    catchError(buildResult: 'SUCCESS', catchInterruptions: true, message: 'Oops! it will be fixed in feature release', stageResult: 'UNSTABLE') {
                    sh 'npm run coverage'
                    }
                }
                publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, icon: '', keepAll: true, reportDir: 'coverage/Icov-report', reportFiles: 'index.html', reportName: 'Code Coverage HTML Report', reportTitles: '', useWrapperFileDirectly: true])
            }
        }  
    }
}

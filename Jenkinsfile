pipeline {
    agent any
    triggers {
	githubPush()
    }
    tools {
	nodejs 'nodejs-v22.22.1'
   }
	
    stages {
        stage('Insatlling Dependencies') {
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
                sh 'npm test'
            }
        }  
    }
}

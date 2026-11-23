
pipeline {
    agent any

    triggers {
        githubPush()
    }

    options {
        timestamps()
        disableConcurrentBuilds()
        skipDefaultCheckout(false)
    }

    tools {
        nodejs 'nodejs-v22.22.1'
    }

    environment {
        MONGO_URI = 'mongodb+srv://cluster0.tpmafor.mongodb.net/superData?authSource=admin'
        SONAR_SCANNER_HOME = tool 'sonarqube-scanner-6104477'
    }

    stages {

        stage('Install Dependencies') {
            steps {
                sh 'npm ci --no-audit'
            }
        }

        stage('Dependency Scanning') {
            parallel {

                stage('NPM Security Audit') {
                    steps {
                        sh 'npm audit --audit-level=critical'
                    }
                }

                stage('OWASP Dependency Check') {
                    steps {
                        dependencyCheck(
                            odcInstallation: 'OWASP-DepCheck-10',
                            nvdCredentialsId: 'nvd-api-key',
                            additionalArguments: '''
                                --scan .
                                --out .
                                --format ALL
                                --prettyPrint
                                --disableYarnAudit
                                --suppression ./dependency-check-suppressions.xml
                            '''
                        )

                        dependencyCheckPublisher(
                            pattern: 'dependency-check-report.xml',
                            failedTotalCritical: 1,
                            stopBuild: true
                        )
                    }

                    post {
                        always {
                            junit(
                                testResults: 'dependency-check-junit.xml',
                                allowEmptyResults: true
                            )

                            publishHTML(target: [
                                reportName: 'Dependency HTML Report',
                                reportDir: '.',
                                reportFiles: 'dependency-check-report.html',
                                keepAll: true,
                                alwaysLinkToLastBuild: true,
                                allowMissing: true
                            ])
                        }
                    }
                }
            }
        }

        stage('Unit Testing') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'Mongodb_Creden',
                        usernameVariable: 'MONGO_USERNAME',
                        passwordVariable: 'MONGO_PASSWORD'
                    )
                ]) {
                    sh 'npm test'
                }
            }

            post {
                always {
                    junit(
                        testResults: 'test-results.xml',
                        allowEmptyResults: false
                    )
                }
            }
        }

        stage('Code Coverage') {
            steps {
                catchError(
                  buildResult: 'SUCCESS',
                  catchInterruptions: true,
                  message: 'Oops! It will be fixed in a feature release',
                  stageResult: 'UNSTABLE'
                ) {
                     withCredentials([
                     usernamePassword(
                     credentialsId: 'Mongodb_Creden',
                     usernameVariable: 'MONGO_USERNAME',
                     passwordVariable: 'MONGO_PASSWORD'
                )
            ]) {
                sh 'npm run coverage'
            }

            sh 'test -s coverage/lcov.info'
        }
    }

    post {
        always {
            publishHTML(target: [
                reportName: 'Code Coverage HTML Report',
                reportDir: 'coverage/lcov-report',
                reportFiles: 'index.html',
                keepAll: true,
                alwaysLinkToLastBuild: true,
                allowMissing: true
            ])
        }
    }
}
        stage('SAST - SonarQube') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    withSonarQubeEnv('sonarqube-server') {
                        sh '''
                            "$SONAR_SCANNER_HOME/bin/sonar-scanner" \
                                -Dsonar.projectKey=Solar-System-Project \
                                -Dsonar.sources=. \
                                -Dsonar.exclusions=node_modules/**,coverage/** \
                                -Dsonar.javascript.lcov.reportPaths=coverage/lcov.info
                        '''
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully.'
        }

        unstable {
            echo 'Pipeline completed with unstable results.'
        }

        failure {
            echo 'Pipeline failed. Check the failing stage.'
        }

        always {
            echo 'Pipeline execution finished.'
        }
    }
}

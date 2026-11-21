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
                    --prettyPrint
                ''', odcInstallation: 'OWASP-DepCheck-10'
            }
        }

    }
}

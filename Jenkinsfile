pipeline {
    agent any
   
    tools {
	nodejs 'nodejs-v22.22.1'
   }
	
    stages {
        stage('VM Node Version') {
            steps {
                sh '''
                    node -v
                    npm -v
                '''
            }
        }
    }
}

pipeline {
    agent any
   
    tools {
	nodejs 'nodejs-v22.22.1'
   }
	
    stages {
        stage('Insatlling Dependencies') {
            steps {
                sh 'npm install --no-audit'
            }
        }
    }
}

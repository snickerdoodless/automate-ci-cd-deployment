pipeline {
    agent { label 'slave-1' } 

    stages {
        stage('Test Ping') {
            steps {
                sh 'ping -c 4 master || { echo "Ping failed!"; exit 1; }'
            }
        }
    }
}

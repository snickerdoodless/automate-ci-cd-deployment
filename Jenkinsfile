pipeline {
    agent { label 'slave-1' } 

    environment {
        APP_IMAGE = 'rall4/flaskapp'
        MYSQL_IMAGE = 'rall4/mysql'
    }

    stages {
        stage('Initialize ID') {
            steps {  
                sh 'whoami && groups'
            }
        }

        stage('Cloning Projects') {
            steps {
                dir('Projects') {
                    git branch: 'main', url: 'https://github.com/snickerdoodless/fastfood-flaskapp'
                }

                dir('Projects') {
                    git branch: 'main', url: 'https://github.com/snickerdoodless/automate-ci-cd-deployment.git'
                }
            }
        }

        stage('Building Flask App Image') {
            steps {
                script {
                    dir('Projects/automate-ci-cd-deployment/app') {
                        docker.build(APP_IMAGE, "../../fastfood-flaskapp/flask-fastfood-app:./")
                    }
                }
            }
        }

        stage('Building MySQL Image') {
            steps {
                script {
                    dir('Projects/automate-ci-cd-deployment/mysql') {
                        docker.build(MYSQL_IMAGE, '.')
                    }
                }
            }
        }

        stage('Tagging Images') {
            steps {
                script {
                    docker.tag(APP_IMAGE, "${APP_IMAGE}:latest")
                    docker.tag(APP_IMAGE, "${APP_IMAGE}:${env.BUILD_TIMESTAMP}")
                    docker.tag(MYSQL_IMAGE, "${MYSQL_IMAGE}:latest")
                    docker.tag(MYSQL_IMAGE, "${MYSQL_IMAGE}:${env.BUILD_TIMESTAMP}")
                    
                    echo "Tagged APP_IMAGE: ${APP_IMAGE}:${env.BUILD_TIMESTAMP}"
                    echo "Tagged MYSQL_IMAGE: ${MYSQL_IMAGE}:${env.BUILD_TIMESTAMP}"
                }
            }
        }
    }
}

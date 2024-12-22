pipeline {
    agent { label 'slave-1' } 

    environment {
        APP_IMAGE = 'rall4/flaskapp'
        MYSQL_IMAGE = 'rall4/mysql'
    }

    stages {
        stage('Initialize ID') {
            steps {  
                //debug
                sh 'whoami && groups'
            }
        }

        stage('Cloning Projects') {
            steps {
                dir('Projects/flask') {
                    git branch: 'main', url: 'https://github.com/snickerdoodless/fastfood-flaskapp'
                }

                dir('Projects/ci-cd') {
                    git branch: 'pipeline', url: 'https://github.com/snickerdoodless/automate-ci-cd-deployment.git'
                }   
            }
        }

        stage('Prepare Build Context') {
            steps {
                // use absolute path to avoid error
                // debugging: 
                sh 'ls -l /opt/jenkins-slave/workspace/automate-deployment/Projects/flask/flask-fastfood-app/'
                sh 'ls -l /opt/jenkins-slave/workspace/automate-deployment/Projects/ci-cd/automate-ci-cd-deployment/app/'

                // executing build context
                sh 'cp -r /opt/jenkins-slave/workspace/automate-deployment/Projects/flask/flask-fastfood-app/* /opt/jenkins-slave/workspace/automate-deployment/Projects/ci-cd/automate-ci-cd-deployment/app/'
                sh 'cp /opt/jenkins-slave/workspace/automate-deployment/Projects/flask/flask-fastfood-app/feane.sql /opt/jenkins-slave/workspace/automate-deployment/Projects/ci-cd/automate-ci-cd-deployment/db/'
            }
        }


        stage('Building Flask App Image') {
            steps {
                script {
                    dir('Projects/automate-ci-cd-deployment/app') {
                        docker.build(APP_IMAGE, '.')
                    }
                }
            }
        }

        stage('Building MySQL Image') {
            steps {
                script {
                    dir('Projects/automate-ci-cd-deployment/db') {
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

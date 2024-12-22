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
                // Use absolute path to avoid errors
                // Debugging: 
                sh 'ls -l /opt/jenkins-slave/workspace/automate-deployment/Projects/flask/flask-fastfood-app/'
                sh 'ls -l /opt/jenkins-slave/workspace/automate-deployment/Projects/ci-cd/app/'
                sh 'ls -l /opt/jenkins-slave/workspace/automate-deployment/Projects/ci-cd/db/'

                // Executing build context
                sh 'cp -r /opt/jenkins-slave/workspace/automate-deployment/Projects/flask/flask-fastfood-app/* /opt/jenkins-slave/workspace/automate-deployment/Projects/ci-cd/app/'
                sh 'cp /opt/jenkins-slave/workspace/automate-deployment/Projects/flask/flask-fastfood-app/feane.sql /opt/jenkins-slave/workspace/automate-deployment/Projects/ci-cd/db/'
            }
        }

        stage('Building Flask App Image') {
            steps {
                script {
                    dir('Projects/ci-cd/app') {
                        docker.build(APP_IMAGE, '.')
                    }
                }
            }
        }

        stage('Building MySQL Image') {
            steps {
                script {
                    dir('Projects/ci-cd/db') {
                        docker.build(MYSQL_IMAGE, '.')
                    }
                }
            }
        }

        stage('Tagging & Pushing to Registry') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', 
                                                    usernameVariable: 'DOCKER_USER', 
                                                    passwordVariable: 'DOCKER_PASS')]) {
                        sh """
                            echo "${DOCKER_PASS}" | docker login -u "${DOCKER_USER}" --password-stdin
                        """

                         // Reformat BUILD_TIMESTAMP
                        def sanitizedTimestamp = (env.BUILD_TIMESTAMP ?: "manual")
                            .replaceAll(' ', '_') // Replace space with underscore
                            .replaceAll(':', '-')  // Remove colons
                            .replaceAll('[^a-zA-Z0-9_.-]', '') // Remove invalid characters

                        def appImage = docker.image("${APP_IMAGE}")
                        appImage.tag("flaskapp-latest") 
                        appImage.push("flaskapp-latest")

                        def appTagTimestamp = env.BUILD_TIMESTAMP ?: "manual"
                        appImage.tag("flaskapp-${appTagTimestamp}") 
                        appImage.push("flaskapp-${appTagTimestamp}")

                        echo "Successfully pushed Flask App image with tags: flaskapp-latest and flaskapp-${appTagTimestamp}"

                        def mysqlImage = docker.image("${MYSQL_IMAGE}")
                        mysqlImage.tag("mysql-latest") 
                        mysqlImage.push("mysql-latest")

                        def mysqlTagTimestamp = env.BUILD_TIMESTAMP ?: "manual"
                        mysqlImage.tag("mysql-${mysqlTagTimestamp}")
                        mysqlImage.push("mysql-${mysqlTagTimestamp}")

                        echo "Successfully pushed MySQL image with tags: mysql-latest and mysql-${mysqlTagTimestamp}"
                    }
                }
            }

            post {
                always {
                    // Ensure logout after the job
                    sh "docker logout"
                }
            }
        }

    }
}

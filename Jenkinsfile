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
                        // Docker login
                        sh """
                            echo "${DOCKER_PASS}" | docker login -u "${DOCKER_USER}" --password-stdin
                        """

                        // Push Flask App Image
                        def appImage = docker.image("${APP_IMAGE}")
                        appImage.push("latest")
                        
                        def appTagTimestamp = env.BUILD_TIMESTAMP ?: "manual"
                        appImage.push(appTagTimestamp)
                        echo "Successfully pushed Flask App image with tags: latest and ${appTagTimestamp}"

                        // Push MySQL Image
                        def mysqlImage = docker.image("${MYSQL_IMAGE}")
                        mysqlImage.push("latest")

                        def mysqlTagTimestamp = env.BUILD_TIMESTAMP ?: "manual"
                        mysqlImage.push(mysqlTagTimestamp)
                        echo "Successfully pushed MySQL image with tags: latest and ${mysqlTagTimestamp}"
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
}

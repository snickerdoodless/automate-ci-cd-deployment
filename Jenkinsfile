pipeline {
    agent { label 'slave-1' }

    environment {
        DOCKER_REPO = 'rall4/myflaskapp'
        DEPLOY_SERVER = '172.31.80.196'
        // Add version tagging strategy
        APP_VERSION = sh(script: 'date +%Y%m%d_%H%M%S', returnStdout: true).trim()
        MYSQL_ROOT_PASSWORD = credentials('mysql-root-password')
        DB_PASSWORD = credentials('db-password')
        SLACK_CHANNEL = '#flask-project'
    }

    stages {
        stage('Validate Environment') {
            steps {
                script {
                    sh '''
                        whoami && groups
                        docker --version
                        git --version
                    '''
                }
            }
        }

        stage('Checkout') {
            parallel {
                stage('Flask App') {
                    steps {
                        dir('Projects/flask') {
                            git(
                                url: 'https://github.com/snickerdoodless/fastfood-flaskapp',
                                branch: 'main',
                                changelog: true
                            )
                        }
                    }
                }
                stage('CI/CD Config') {
                    steps {
                        dir('Projects/ci-cd') {
                            git(
                                url: 'https://github.com/snickerdoodless/automate-ci-cd-deployment.git',
                                branch: 'pipeline',
                                changelog: true
                            )
                        }
                    }
                }
            }
        }

        stage('Prepare Build Context') {
            steps {
                script {
                    try {
                        sh '''
                            cp -r Projects/flask/flask-fastfood-app/* Projects/ci-cd/app/
                            cp Projects/flask/flask-fastfood-app/feane.sql Projects/ci-cd/db/
                        '''
                    } catch (Exception e) {
                        error "Failed to prepare build context: ${e.message}"
                    }
                }
            }
        }

        stage('Build and Test') {
            parallel {
                stage('Flask App') {
                    steps {
                        script {
                            dir('Projects/ci-cd/app') {
                                docker.build(
                                    "${DOCKER_REPO}:flaskapp-${APP_VERSION}",
                                    "--build-arg app=flaskapp --no-cache ."
                                )
                            }
                        }
                    }
                }
                stage('MySQL') {
                    steps {
                        script {
                            dir('Projects/ci-cd/db') {
                                docker.build(
                                    "${DOCKER_REPO}:mysql-${APP_VERSION}",
                                    "--build-arg app=mysql --no-cache ."
                                )
                            }
                        }
                    }
                }
            }
        }

        stage('Push Images') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'docker-hub-credentials') {
                        try {
                            // Push Flask App
                            def flaskImage = docker.image("${DOCKER_REPO}:flaskapp-${APP_VERSION}")
                            flaskImage.push()
                            flaskImage.push('flaskapp-latest')

                            // Push MySQL
                            def mysqlImage = docker.image("${DOCKER_REPO}:mysql-${APP_VERSION}")
                            mysqlImage.push()
                            mysqlImage.push('mysql-latest')
                        } finally {
                            // Clean up images
                            sh """
                                # Remove versioned images
                                docker rmi ${DOCKER_REPO}:flaskapp-${APP_VERSION} || true
                                docker rmi ${DOCKER_REPO}:mysql-${APP_VERSION} || true
                                
                                # Remove latest tagged images
                                docker rmi ${DOCKER_REPO}:flaskapp-latest || true
                                docker rmi ${DOCKER_REPO}:mysql-latest || true
                                
                                # Clean up any dangling images, cache & etc.
                                docker system prune -a -f
                            """
                        }
                    }
                }
            }
            post {
                always {
                    sh 'docker logout'
                }
            }
        }

        stage('Deploy') {
            stages {
                stage('Setup Network') {
                    steps {
                        sshagent(['deploy-server-credentials']) {
                            sh '''
                                ssh ${DEPLOY_SERVER} '
                                    # Stop and remove existing containers to prepare network
                                    docker rm -f flask-container mysql-container || true

                                    docker network rm -f feane_network || true
                                    docker network create --subnet=192.168.10.0/24 feane_network
                                '
                            '''
                        }
                    }
                }

                stage('Deploy Containers') {
                    steps {
                        sshagent(['deploy-server-credentials']) {
                            sh '''
                                ssh ${DEPLOY_SERVER} "
                                    # Pull latest images
                                    docker rmi -f ${DOCKER_REPO}:flaskapp-latest
                                    docker pull ${DOCKER_REPO}:flaskapp-latest
                                    docker rmi -f ${DOCKER_REPO}:mysql-latest
                                    docker pull ${DOCKER_REPO}:mysql-latest

                                    # Start MySQL
                                    docker run -d --name mysql-container \\
                                        -e MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD} \\
                                        -e MYSQL_DATABASE=feane \\
                                        -p 3306:3306 \\
                                        --net feane_network --ip 192.168.10.5 \\
                                        --restart unless-stopped \\
                                        ${DOCKER_REPO}:mysql-latest

                                    # Wait for MySQL to be ready
                                    sleep 30

                                    # Start Flask App
                                    docker run -d --name flask-container \\
                                        --link mysql-container:mysql \\
                                        -e DB_HOST=192.168.10.5 \\
                                        -e DB_NAME=feane \\
                                        -e DB_USER=feane \\
                                        -e DB_PASS=${DB_PASSWORD} \\
                                        -p 5000:5000 \\
                                        --net feane_network --ip 192.168.10.10 \\
                                        --restart unless-stopped \\
                                        ${DOCKER_REPO}:flaskapp-${APP_VERSION}

                                    # Verify containers are running
                                    docker ps | grep -E 'flask-container|mysql-container'
                                "
                            '''
                        }
                    }
                }
            }
        }
    }

    post {
        success {
            slackSend(
                channel: "${SLACK_CHANNEL}",
                color: '#36A64F',
                message: """
                    ✅ Deployment Successful
                    Build Number: ${env.BUILD_NUMBER}
                    Environment: Development
                    Version: ${APP_VERSION}
                    Images:
                    • Flask App: ${DOCKER_REPO}:flaskapp-${APP_VERSION}
                    • MySQL: ${DOCKER_REPO}:mysql-${APP_VERSION}
                """
            )
        }
        failure {
            slackSend(
                channel: "${SLACK_CHANNEL}",
                color: '#FF0000',
                message: """
                    ❌ Deployment Failed
                    Build Number: ${env.BUILD_NUMBER}
                    Environment: Development
                    Version: ${APP_VERSION}
                    Stage: ${currentBuild.displayName}
                    Check logs: ${env.BUILD_URL}
                """
            )
        }
        always {
            cleanWs()
        }
    }

}

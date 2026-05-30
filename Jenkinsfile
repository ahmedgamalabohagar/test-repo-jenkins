def dockerImage = ''

pipeline {
    agent any

    environment {
        REGISTRY      = 'docker.io'
        REGISTRY_CRED = 'dockerhub-cred'
        IMAGE_NAME    = 'ahmedabohagar/my-app'
        IMAGE_TAG     = "1.0.${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                echo "Branch : ${GIT_BRANCH}"
                echo "Commit : ${GIT_COMMIT}"
            }
        }

        stage('Build Image') {
            steps {
                script {
                    echo "Building Docker image ${IMAGE_NAME}:${IMAGE_TAG}"
                    // شيلنا كلمة def من هنا لأننا عرفناها فوق خالص
                    dockerImage = docker.build("${IMAGE_NAME}:${IMAGE_TAG}")
                }
            }
        }

        stage('Test Image') {
            steps {
                script {
                    // run the container and test it
                    dockerImage.inside {
                        sh 'echo "Container is running fine"'
                        sh 'python --version'
                    }
                }
            }
        }

        stage('Push Image') {
            steps {
                script {
                    docker.withRegistry("https://${REGISTRY}", "${REGISTRY_CRED}") {
                        // push with build number tag
                        dockerImage.push("${IMAGE_TAG}")
                        // push as latest
                        dockerImage.push('latest')
                    }
                }
            }
        }

        stage('Deploy') {
            when {
                branch 'main'
            }
            steps {
                script {
                    sh """
                        # stop old container if running
                        docker stop my-app || true
                        docker rm   my-app || true

                        # run new container
                        docker run -d \
                            --name my-app \
                            -p 8080:8080 \
                            --restart always \
                            ${IMAGE_NAME}:${IMAGE_TAG}

                        echo "Container deployed successfully"
                    """
                }
            }
        }

        stage('Verify Deployment') {
            when {
                branch 'main'
            }
            steps {
                script {
                    sleep(time: 10, unit: 'SECONDS')

                    def status = sh(
                        script: 'docker inspect -f {{.State.Running}} my-app',
                        returnStdout: true
                    ).trim()

                    if (status != 'true') {
                        error('Container is not running after deployment')
                    }

                    echo "Container is running successfully"
                }
            }
        }
    }

    post {
        success {
            echo "SUCCESS - ${IMAGE_NAME}:${IMAGE_TAG} deployed"
        }
        failure {
            echo "FAILURE - Build #${BUILD_NUMBER} failed"
        }
        always {
            // clean up local images to save disk space
            sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG} || true"
            cleanWs()
        }
    }
}

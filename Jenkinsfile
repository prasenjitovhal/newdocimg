pipeline {
    agent any

    environment {
        APP_NAME     = "myapp"
        IMAGE_NAME   = "prasenjitovhal/myapp"   // CHANGE THIS
        DEV_SERVER   = "ec2-user@13.127.30.198"
        STAGE_SERVER = "ec2-user@13.232.187.168"
        PROD_SERVER  = "ec2-user@65.1.134.248"
        DOCKER_CREDS = "dockerhub-creds"
        TAG          = "${BRANCH_NAME.toLowerCase()}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: DOCKER_CREDS,
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh '''
                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t ${IMAGE_NAME}:${TAG} .
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                sh '''
                docker push ${IMAGE_NAME}:${TAG}
                docker tag ${IMAGE_NAME}:${TAG} ${IMAGE_NAME}:latest
                docker push ${IMAGE_NAME}:latest
                '''
            }
        }

        stage('Deploy') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'dev') {
                        deploy(DEV_SERVER, 8081)
                    } else if (env.BRANCH_NAME == 'stage') {
                        deploy(STAGE_SERVER, 8082)
                    } else if (env.BRANCH_NAME == 'main' || env.BRANCH_NAME == 'Main') {
                        deploy(PROD_SERVER, 80)
                    }
                }
            }
        }
    }
}

def deploy(server, port) {
    sh """
    ssh -o StrictHostKeyChecking=no ${server} '
        docker pull ${IMAGE_NAME}:${TAG}
        docker rm -f ${APP_NAME} || true
        docker run -d -p ${port}:80 --name ${APP_NAME} ${IMAGE_NAME}:${TAG}
    '
    """
}

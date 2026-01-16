pipeline {
    agent any

    environment {
        APP_NAME     = "myapp"
        IMAGE_NAME   = "mydockerhubusername/myapp"   // change this
        DEV_SERVER   = "ec2-user@13.127.30.198"
        STAGE_SERVER = "ec2-user@13.232.187.168"
        PROD_SERVER  = "ec2-user@65.1.134.248"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                docker build -t ${IMAGE_NAME}:${BRANCH_NAME} .
                """
            }
        }

        stage('Push Docker Image') {
            steps {
                sh """
                docker tag ${IMAGE_NAME}:${BRANCH_NAME} ${IMAGE_NAME}:latest
                docker push ${IMAGE_NAME}:${BRANCH_NAME}
                docker push ${IMAGE_NAME}:latest
                """
            }
        }

        stage('Deploy') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'dev') {
                        deploy(DEV_SERVER, 8081)
                    }
                    else if (env.BRANCH_NAME == 'stage') {
                        deploy(STAGE_SERVER, 8082)
                    }
                    else if (env.BRANCH_NAME == 'main') {
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
        docker pull ${IMAGE_NAME}:${BRANCH_NAME}
        docker rm -f ${APP_NAME} || true
        docker run -d \
          -p ${port}:80 \
          --name ${APP_NAME} \
          ${IMAGE_NAME}:${BRANCH_NAME}
    '
    """
}

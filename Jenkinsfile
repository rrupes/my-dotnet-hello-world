pipeline {
    agent any 

parameters {
    choice(name: 'ENV', choices: ['UAT', 'PROD'], description: 'Select environment')

}


    environment {
        DOCKER_IMAGE = "rupesh9136/net_image:${BUILD_NUMBER}"
         PROD_HOST = "ec2-user@cn.tg.com"
         AWS_KEY = "awskey"
    }

    stages {

        stage('Code Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/rrupes/my-dotnet-hello-world.git'
            }
        }

        stage('Creating Docker Image') {
            steps {
                sh """
                    docker build -t ${DOCKER_IMAGE} .
                """
            }
        }

        stage('Login to DockerHub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockercred',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh """
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push ${DOCKER_IMAGE}
                    """
                }
            }
        }

 stage('Deploy - UAT / PROD') {
            steps {
                script {

                    if (params.ENV == "UAT") {
                        echo "Deploying to UAT (local Jenkins EC2)..."
                        
                        sh """
                            docker rm -f netapp || true
                            docker pull ${DOCKER_IMAGE}
                            docker run -d --name netapp -p 8888:80 ${DOCKER_IMAGE}
                        """
                    }

                    if (params.ENV == "PROD") {
                        echo "Deploying to PROD EC2 over SSH..."

                        withCredentials([sshUserPrivateKey(
                            credentialsId: AWS_KEY,
                            keyFileVariable: 'SSH_KEY'
                        )]) {
                            sh """
                                ssh -o StrictHostKeyChecking=no -i "$SSH_KEY" ${PROD_HOST} '
                                    docker rm -f netapp || true
                                    docker pull ${DOCKER_IMAGE}
                                    docker run -d --name netapp -p 8811:80 ${DOCKER_IMAGE}
                                '
                            """
                        }
                    }
                }
            }
        }
    }

    post {
        success {
            echo "Deployment to ${params.ENV} completed successfully!"
        }
        failure {
            echo "Deployment failed. Check logs."
        }
    }
}


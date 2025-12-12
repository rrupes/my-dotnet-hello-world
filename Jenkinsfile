pipeline {
    agent any 

    environment {
        IMAGE = "net_image:${BUILD_NUMBER}"
        DOCKER_IMAGE = "rupesh9136/net_image:${BUILD_NUMBER}"
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

        stage('Deploy on Same EC2') {
            steps {
                sh """
                    docker rm -f netapp || true
                    docker pull ${DOCKER_IMAGE}
                    docker run -d --name netapp -p 8888:80 ${DOCKER_IMAGE}
                """
            }
        }
    }

    post {
        success {
            echo "Deployment Successful! App running on port 8888"
        }
    }
}

pipeline {
    agent none

    environment {
        IMAGE_NAME = 'webapp'
        IMAGE_TAG = 'v2.5'
        DOCKERHUB = credentials('dockerhub')
        DOCKER_INTERNAL_IP_ADDRESS = '172.17.0.1'
        PORT = '80'
    }

    stages {
        stage('build') {
            agent any

            steps {
                echo '********** Build **********'
                sh '''
                    set -e
                    docker build -t $DOCKERHUB_USR/$IMAGE_NAME:$IMAGE_TAG .
                '''
            }
        }

        stage('test') {
            agent any

            steps {
                echo '********** Test **********'
                sh ''' 
                    set -e
                    docker rm -f $IMAGE_NAME || echo 'Container $IMAGE_NAME does not exist'
                    docker run -d --name $IMAGE_NAME -p $PORT:5000 -e PORT=5000 $DOCKERHUB_USR/$IMAGE_NAME:$IMAGE_TAG
                    sleep 5s

                    curl http://$DOCKER_INTERNAL_IP_ADDRESS:$PORT | grep -q 'Hello world!'

                    echo 'Cleaning the resources'
                    docker stop $IMAGE_NAME
                    docker rm $IMAGE_NAME
                '''
            }
        }

        stage('release') {
            agent any

            steps {
                echo '********** Release **********'
                sh ''' 
                    set -e
                    echo $DOCKERHUB_PSW | docker login -u $DOCKERHUB_USR --password-stdin
                    docker push $DOCKERHUB_USR/$IMAGE_NAME:$IMAGE_TAG
                '''
            }
        }

        stage('deploy staging') {
            agent any

            when { 
                expression { env.BRANCH_NAME == 'origin/master' }
            }

            environment {
                VM_IP_ADDRESS = '34.207.123.147'
            }

            steps {
                echo '********** Deploy staging **********'
                sshagent (credentials: ['aws_secret_key']) {
                    sh '''
                        set -e
                        ssh -o StrictHostKeyChecking=no -t ubuntu@$VM_IP_ADDRESS "
                            echo $DOCKERHUB_PSW | docker login -u $DOCKERHUB_USR --password-stdin && \
                            docker pull $DOCKERHUB_USR/$IMAGE_NAME:$IMAGE_TAG && \
                            docker container rm -f webapp || echo 'No existing container' && \
                            docker run -d -p 80:5000 -e PORT=5000 --name=webapp $DOCKERHUB_USR/$IMAGE_NAME:$IMAGE_TAG
                        "
                    '''
                }
            }
        }

        stage('deploy prod') {
            agent any

            when { 
                expression { env.BRANCH_NAME == 'origin/master' }
            }

            environment {
                VM_IP_ADDRESS = '3.95.202.51'
            }

            steps {
                echo '********** Deploy prod **********'
                sshagent (credentials: ['aws_secret_key']) {
                    sh '''
                        set -e
                        ssh -o StrictHostKeyChecking=no -t ubuntu@$VM_IP_ADDRESS "
                            echo $DOCKERHUB_PSW | docker login -u $DOCKERHUB_USR --password-stdin && \
                            docker pull $DOCKERHUB_USR/$IMAGE_NAME:$IMAGE_TAG && \
                            docker container rm -f webapp || echo 'No existing container' && \
                            docker run -d -p 80:5000 -e PORT=5000 --name=webapp $DOCKERHUB_USR/$IMAGE_NAME:$IMAGE_TAG
                        "
                    '''
                }
            }
        }
    }
}

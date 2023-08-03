pipeline {
    agent any
    environment {
        AWS_ACCOUNT = '352264280014'
        AWS_REGION = 'ap-northeat-2'
        IMAGE_NAME = 'test'
        IMAGE_TAG = 'latest'
    }
    stages {
	stage('Create Dockerfile') {
            steps {
                sh '''
                    echo "FROM ubuntu" > Dockerfile
                    echo "RUN apt update" >> Dockerfile
                    echo "RUN apt install -y apache2" >> Dockerfile
                    echo "RUN apt install -y apache2-utils" >> Dockerfile
                    echo "RUN apt clean" >> Dockerfile
                    echo "EXPOSE 80" >> Dockerfile
                    echo 'CMD [ “apache2ctl”, “-D”, “FOREGROUND”]' >> Dockerfile
                '''
            }
        }

        stage('Build') {
            steps {
                sh 'docker build -t $IMAGE_NAME:$IMAGE_TAG .'
            }
        }
        stage('Push') {
            steps {
                    sh 'docker tag $IMAGE_NAME:$IMAGE_TAG public.ecr.aws/t6r5u3y4/$IMAGE_NAME:$IMAGE_TAG'
                    sh 'docker push public.ecr.aws/t6r5u3y4/$IMAGE_NAME:$IMAGE_TAG'
                }
            }
        }
    }


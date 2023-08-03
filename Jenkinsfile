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
                withAWS(region: "$AWS_REGION", credentials: 'aws-credentials') {
                    sh "eval \$(aws ecr get-login --no-include-email --region $AWS_REGION)"
                    sh "docker tag $IMAGE_NAME:$IMAGE_TAG $AWS_ACCOUNT.dkr.ecr.$AWS_REGION.amazonaws.com/$IMAGE_NAME:$IMAGE_TAG"
                    sh "docker push $AWS_ACCOUNT.dkr.ecr.$AWS_REGION.amazonaws.com/$IMAGE_NAME:$IMAGE_TAG"
                }
            }
        }
    }
}

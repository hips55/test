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
                sh 'docker build -t 352264280014.dkr.ecr.ap-northeast-2.amazonaws.com/test .'
            }
        }
        stage('Push') {
            steps {docker.withRegistry("https://${ECR_PATH}", "ecr:${REGION}:AWSCredentials") {
                            def image = docker.build("${ECR_PATH}/${ECR_IMAGE}:${env.BUILD_NUMBER}")
                            image.push()
                        }
                            }
            }
        }
    }


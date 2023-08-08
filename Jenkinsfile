pipeline {
    agent any
    environment {
        AWS_ACCOUNT = '352264280014'
        AWS_REGION = 'ap-northeast-2'
        IMAGE_NAME = 'test'
        IMAGE_TAG = 'latest'
        ECR_PATH = "${AWS_ACCOUNT}.dkr.ecr.${AWS_REGION}.amazonaws.com"
        CONTAINER_PORT = 8080 // 변경 가능한 포트 번호 입력
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
                    echo 'CMD [ "apache2ctl", "-D", "FOREGROUND"]' >> Dockerfile
                '''
            }
        }

        stage('Build') {
            steps {
                sh 'docker build -t ${ECR_PATH}/${IMAGE_NAME}:${IMAGE_TAG} .'
            }
        }
        stage('Push') {
            steps {
                script {
                    try {
                        docker.withRegistry("https://${ECR_PATH}", "ecr:${AWS_REGION}:aws-credentials") {
                            def image = docker.build("${ECR_PATH}/${IMAGE_NAME}:${env.BUILD_NUMBER}")
                            image.push()
                        }
                        echo 'Remove Deploy Files'
                        sh "rm -rf /var/lib/jenkins/workspace/${env.JOB_NAME}/*"
                        env.dockerBuildResult = true
                    } catch (error) {
                        print(error)
                        echo 'Remove Deploy Files'
                        sh "rm -rf /var/lib/jenkins/workspace/${env.JOB_NAME}/*"
                        env.dockerBuildResult = false
                        currentBuild.result = 'FAILURE'
                    }
                }
            }
        }
        stage('Stop and Remove Previous Container') {
            steps {
                script {
                    def existingContainers = sh(script: 'docker ps -q', returnStdout: true).trim()
                    if (existingContainers) {
                        sh 'docker stop $(docker ps -q) && docker rm $(docker ps -a -q)'
                    } else {
                        echo 'No existing containers to stop and remove.'
                    }
                }
            }
        }
        stage('Run') {
            steps {
                script {
                    sh "docker run -d -p 80:80 ${ECR_PATH}/${IMAGE_NAME}:${IMAGE_TAG}"
                }
            }
        }
        stage('Deploy HTML') {
            steps {
                sh '''
                    docker exec -i $(docker ps -q) sh -c "echo '<html><body><h1>FreshSales</h1></body></html>' > /var/www/html/index.html"
                '''
            }
        }
    }
}


pipeline {
    agent any
    environment {
        AWS_ACCOUNT = '352264280014'
        AWS_REGION = 'ap-northeast-2'
        IMAGE_NAME = 'petclinic'
        IMAGE_TAG = 'latest'
        ECR_PATH = "${AWS_ACCOUNT}.dkr.ecr.${AWS_REGION}.amazonaws.com"
        CONTAINER_PORT = 8080
    }
    stages {
        stage('Build PetClinic') {
            steps {
                sh 'git clone https://github.com/spring-projects/spring-petclinic.git'
                sh 'cd spring-petclinic && ./mvnw package'
            }
        }

        stage('Create Dockerfile') {
            steps {
                sh '''
                    echo "FROM openjdk:11-jre-slim" > Dockerfile
                    echo "COPY spring-petclinic/target/*.jar /app.jar" >> Dockerfile
                    echo "EXPOSE ${CONTAINER_PORT}" >> Dockerfile
                    echo 'CMD ["java", "-jar", "/app.jar"]' >> Dockerfile
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
                    sh "docker run -d -p ${CONTAINER_PORT}:${CONTAINER_PORT} ${ECR_PATH}/${IMAGE_NAME}:${IMAGE_TAG}"
                }
            }
        }

        stage('Deploy HTML') {
            steps {
                sh '''
                    docker exec -i $(docker ps -q) sh -c "echo '<html><body><h1>PetClinic</h1></body></html>' > /app.html"
                '''
            }
        }
    }
}


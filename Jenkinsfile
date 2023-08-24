pipeline {
    agent any
    environment {
        AWS_ACCOUNT = '213899591783'
        AWS_REGION = 'ap-northeast-2'
        IMAGE_NAME = 'jenkins-hhs'
        IMAGE_TAG = 'latest'
        ECR_PATH = "${AWS_ACCOUNT}.dkr.ecr.${AWS_REGION}.amazonaws.com"
        CONTAINER_PORT = 8080
    }
    stages {
        stage('Clean Workspace') {
            steps {
                sh "rm -rf spring-petclinic"
            }
        }

        stage('Build PetClinic') {
            steps {
                sh 'git clone https://github.com/spring-projects/spring-petclinic.git'
                sh 'cd spring-petclinic && ./mvnw clean package -DskipTests'
            }
        }

        stage('Create Dockerfile') {
            steps {
                sh '''
                    echo "FROM openjdk:17" > Dockerfile
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
                        docker.withRegistry("https://${ECR_PATH}", "ecr:${AWS_REGION}:AWSCredentials") {
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

 stage('Push Yaml'){
      steps {
        git url: 'https://github.com/hips55/test.git', branch: "main" , credentialsId: 'hyeonsik2'
        
        sh """
        #!/bin/bash
        cat > deploy.yaml << EOF
        apiVersion: apps/v1
kind: Deployment
metadata:
  name: tomcat-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: was
  template:
    metadata:
      labels:
        app: was
    spec:
      containers:
      - image: ${AWS_ACCOUNT}.dkr.ecr.ap-northeast-2.amazonaws.com/${IMAGE_NAME}:${env.BUILD_NUMBER}
        name: petclinic
        ports:
        - name: http
          containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: petclinic-service
spec:
  selector:
    app: was
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 8080
  type: NodePort
"""
 
        sh '''
        git remote set-url origin git@github.com:hips55/test.git
        git add deploy.yaml
        git commit -m 'yaml for deploy'
        git push -u origin  main
        '''
      }
    }
    }
}


pipeline {
    agent any

    tools {
        maven 'mvn' 
        jdk 'jdk-17'
    }

    environment {
        // Replace with your Docker Hub username and repository name
        DOCKER_HUB_USER = "selfcreated"
        IMAGE_NAME = "java-hello-world"
        REGISTRY = "${DOCKER_HUB_USER}/${IMAGE_NAME}"
        DOCKER_HUB_CRE = credentials('dockerhub-cre')
        DOCKER_HUB_CRE_USR = DOCKER_HUB_CRE.username
        DOCKER_HUB_CRE_PSW = DOCKER_HUB_CRE.password
        
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Docker Build') {
            steps {
                sh "docker build -t ${REGISTRY}:${BUILD_NUMBER} ."
                sh "docker tag ${REGISTRY}:${BUILD_NUMBER} ${REGISTRY}:latest"
            }
        }

        stage('Docker Push') {
            steps {
                sh "echo ${DOCKER_HUB_CRE_PSW} | docker login -u ${DOCKER_HUB_CRE_USR} --password-stdin"
                sh "docker push ${REGISTRY}:${BUILD_NUMBER}"
                sh "docker push ${REGISTRY}:latest"
            }
        }

        stage('Deploy') {
            steps {
                // Stop and remove existing container if it exists
                sh "docker stop ${IMAGE_NAME} || true"
                sh "docker rm ${IMAGE_NAME} || true"
                // Run the new container
                sh "docker run -d --name ${IMAGE_NAME} -p 8080:8080 ${REGISTRY}:latest"
            }
        }

        stage('Cleanup') {
            steps {
                // Optional: Remove local images to save space, but keep the one running if necessary
                // Here we remove the specific build tag but keep latest if we want to run it via 'latest'
                sh "docker rmi ${REGISTRY}:${BUILD_NUMBER} || true"
                sh "docker logout"
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}

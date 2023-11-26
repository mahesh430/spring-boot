pipeline {
    agent {
        docker {
            image 'maven' // Specify the Maven Docker image
            args '-v $HOME/.m2:/root/.m2'// Optional: Mount Maven's .m2 directory to cache dependencies
        }
    }
    environment {
        // Define environment variables
        DOCKERHUB_CREDENTIALS = credentials('docker-creds') // ID of your Docker Hub credentials in Jenkins
        IMAGE_TAG = "mahesh430/complete-cicd:${BUILD_NUMBER}"
        SONAR_URL = "http://sonarqube.infonxt.com:9000/"
        GIT_REPO_NAME = "complete-cicd"
        GIT_USER_NAME = "mahesh430"
    }
    stages {
        stage('Checkout') {
            steps {
                sh 'echo passed'
                // Uncomment the next line to enable git checkout
                // git branch: 'main', url: 'https://github.com/mahesh430/spring-boot.git'
            }
        }
        stage('Build and Test') {
            steps {
                sh 'ls -ltr'
                sh 'mvn clean package'
            }
        }
        stage('Static Code Analysis') {
            steps {
                withCredentials([string(credentialsId: 'Sonarqube', variable: 'SONAR_AUTH_TOKEN')]) {
                    sh 'mvn sonar:sonar -Dsonar.login=$SONAR_AUTH_TOKEN -Dsonar.host.url=${SONAR_URL}'
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${IMAGE_TAG} ."
                }
            }
        }
        stage('Docker Image Scan') {
            steps {
                script {
                    sh "trivy image --exit-code 1 --no-progress ${IMAGE_TAG}"
                }
            }
        }
        stage('Push to Docker Hub') {
            steps {
                script {
                    // Login to Docker Hub
                    sh "echo ${DOCKERHUB_CREDENTIALS_PSW} | docker login --username ${DOCKERHUB_CREDENTIALS_USR} --password-stdin"
                    // Pushing Image to Docker Hub
                    sh "docker push ${IMAGE_TAG}"
                }
            }
        }
        stage('Update Deployment File') {
            steps {
                withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                    sh '''
                        git config user.email "abhishek.xyz@gmail.com"
                        git config user.name "Abhishek Veeramalla"
                        sed -i "s/replaceImageTag/${BUILD_NUMBER}/g" java-maven-sonar-argocd-helm-k8s/spring-boot-app-manifests/deployment.yml
                        git add java-maven-sonar-argocd-helm-k8s/spring-boot-app-manifests/deployment.yml
                        git commit -m "Update deployment image to version ${BUILD_NUMBER}"
                        git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                    '''
                }
            }
        }
    }
}

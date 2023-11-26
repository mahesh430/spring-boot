pipeline {
    agent {
        docker {
            image 'mahesh430/maven-jenkins-agent:latest'
            args '--user root -v /var/run/docker.sock:/var/run/docker.sock' // mount Docker socket to access the host's Docker daemon
        }
    }
    environment {
        // Define environment variables
        DOCKERHUB_CREDENTIALS = credentials('docker-hub-credentials') // ID of your Docker Hub credentials in Jenkins
        IMAGE_TAG = "mahesh430/complete-cicd:${BUILD_NUMBER}"
        SONAR_URL = "http://sonarqube.infonxt.com:9000/"
    }
    stages {
        stage('Checkout') {
            steps {
                echo 'Passed'
                // Uncomment the next line if you need to checkout code from Git
                // git branch: 'main', url: 'https://github.com/mahesh430/spring-boot.git'
            }
        }
        stage('Build and Test') {
            steps {
                sh 'ls -ltr'
                sh 'mvn clean package' // build the project and create a JAR file
            }
        }
        stage('Static Code Analysis') {
            steps {
                withCredentials([string(credentialsId: 'Sonarqube', variable: 'SONAR_AUTH_TOKEN')]) {
                    sh "mvn sonar:sonar -Dsonar.login=$SONAR_AUTH_TOKEN -Dsonar.host.url=${SONAR_URL}"
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
        // stage('Docker Image Scan') {
        //     steps {
        //         script {
        //             // Ensure Trivy is installed and accessible in the Jenkins environment
        //          //   sh "trivy image --exit-code 1 --no-progress ${IMAGE_TAG}"
        //         }
        //     }
        // }
        stage('Push to Docker Hub') {
            steps {
                script {
                    sh "echo ${DOCKERHUB_CREDENTIALS_PSW} | docker login --username ${DOCKERHUB_CREDENTIALS_USR} --password-stdin"
                    sh "docker push ${IMAGE_TAG}"
                }
            }
        }
        stage('Update Deployment File') {
            environment {
                GIT_REPO_NAME = "spring-boot-k8s-helm"
                GIT_USER_NAME = "mahesh430"
            }
            steps {
                withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                    sh '''
                        git config user.email "umamahesh690@gmail.com"
                        git config user.name "Mahesh"
                        sed -i "s/replaceImageTag/${BUILD_NUMBER}/g" deployment.yml
                        git add deployment.yml
                        git commit -m "Update deployment image to version ${BUILD_NUMBER}"
                        git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                    '''
                }
            }
        }
    }
}

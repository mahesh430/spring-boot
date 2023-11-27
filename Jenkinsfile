pipeline {
    agent {
        docker {
            image 'mahesh430/maven-jenkins-docker-agent'
            args '--rm --user root -v /var/run/docker.sock:/var/run/docker.sock' // mount Docker socket to access the host's Docker daemon
        }
    }
    environment {
        // Define environment variables
        DOCKERHUB_CREDENTIALS = credentials('docker-hub-credentials') // ID of your Docker Hub credentials in Jenkins
        IMAGE_TAG = "mahesh430/complete-cicd:${BUILD_NUMBER}"
        SONAR_URL = "http://sonarqube.infonxt.com:9000/"
    }
    stages {
        // stage('Prepare') {
        //     steps {
        //         cleanWs() // Clean the workspace at the beginning of the pipeline
        //     }
        // }
        stage('Checkout') {
            steps {
                script {

                    echo 'Passed'
                    git branch: 'main', url: 'https://github.com/mahesh430/spring-boot.git'
                }
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
                    script {
                        try {
                            sh '''
                                set -e  # Ensure the shell exits immediately if a command exits with a non-zero status

                                # Clone the repository
                                git clone https://github.com/${GIT_USER_NAME}/${GIT_REPO_NAME}.git
                                cd ${GIT_REPO_NAME}

                                # Configure Git user
                                git config user.email "umamahesh690@gmail.com"
                                git config user.name "Mahesh"

                                # Update the deployment file
                                sed -i "s/replaceImageTag/${BUILD_NUMBER}/g" deployment.yml

                                # Add and commit changes
                                git add deployment.yml
                                git commit -m "Update deployment image to version ${BUILD_NUMBER}"

                                # Push the changes
                                git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                            '''
                        } catch (Exception e) {
                            // Handle the error
                            echo "Error occurred during deployment file update: ${e.getMessage()}"
                            throw e  // Rethrow the exception to fail the stage
                        }
                    }
                }
            }
        }
    }
    post {
        always {
            echo 'One way or another, I have finished'
            deleteDir() 
        }
    }
}

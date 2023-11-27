pipeline {
    agent {
        docker {
            image 'mahesh430/maven-jenkins-docker-agent-v1'
          args '--user root --rm -v /var/run/docker.sock:/var/run/docker.sock'
        }
    }
    environment {
        DOCKERHUB_CREDENTIALS = credentials('docker-hub-credentials')
        IMAGE_TAG = "mahesh430/complete-cicd:${BUILD_NUMBER}"
        SONAR_URL = "http://sonarqube.infonxt.com:9000/"
    }
    stages {
        stage('Checkout') {
            steps {
                script {
              //      sh 'sleep 1000'
                    echo 'Passed'
                    git branch: 'main', url: 'https://github.com/mahesh430/spring-boot.git'
                }
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
                            sleep 10000
                                set -e

                               # git clone https://github.com/${GIT_USER_NAME}/${GIT_REPO_NAME}.git
                                cd ${GIT_REPO_NAME}

                                git config user.email "umamahesh690@gmail.com"
                                git config user.name "Mahesh"

                                # Update the deployment file
                                sed -i "s/mahesh430\\/complete-cicd:[0-9]*/mahesh430\\/complete-cicd:${BUILD_NUMBER}/g" deployment.yml

                                git add deployment.yml
                                git commit -m "Update deployment image to version ${BUILD_NUMBER}"

                                git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                            '''
                        } catch (Exception e) {
                            echo "Error occurred during deployment file update: ${e.getMessage()}"
                            throw e
                        }
                    }
                }
            }
        }
    }
    // post {
    //     always {
    //         deleteDir() 
    //     }
    // }
}

pipeline {
    agent {
       docker {
            image 'mahesh430/maven-jenkins-docker-agent-v1'
            args '--user root --rm -v /var/run/docker.sock:/var/run/docker.sock -v ${WORKSPACE}:/app'
        }
    }
    environment {
        DOCKERHUB_CREDENTIALS = credentials('docker-hub-credentials')
        IMAGE_TAG = "mahesh430/complete-cicd:${BUILD_NUMBER}"
        SONAR_URL = "http://3.22.92.56:9000/"
        HELM_CHART_PATH = "helm-deploy"

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
                    sh "docker rmi ${IMAGE_TAG}"
                }
            }
        }
     stage('Update Helm Chart') {
            steps {
                withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                    script {
                        try {
                            sh '''
                                set -e
                                cd ~
                                pwd
                                // rm -rf /var/lib/jenkins/workspace/complete-cicd/${GIT_REPO_NAME}

                                git clone https://github.com/${GIT_USER_NAME}/${GIT_REPO_NAME}.git
                                cd ${GIT_REPO_NAME}

                                # Update the Helm chart values file
                                sed -i "s|repository: mahesh430/complete-cicd.*|repository: mahesh430/complete-cicd\n  tag: ${BUILD_NUMBER}|g" ${HELM_CHART_PATH}/values.yaml

                                git config user.email "umamahesh690@gmail.com"
                                git config user.name "Mahesh"
                                git add ${HELM_CHART_PATH}/values.yaml
                                git commit -m "Update Helm chart with image version ${BUILD_NUMBER}"
                                git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                            '''
                        } catch (Exception e) {
                            echo "Error occurred during Helm chart update: ${e.getMessage()}"
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

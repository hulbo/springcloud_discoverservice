pipeline {
    agent any
    tools {
        maven "Maven_3.9.9"
    }

    environment {
        DOCKER_HUL_ID = "hulbo"
        IMAGE_NAME = "sc_discoverservice"
        PORT = "8761:8761"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Set Spring Profile') {
            steps {
                script {
                    def branch = sh(script: "git branch -r --contains HEAD | grep origin/ | head -n 1 | sed 's@origin/@@'", returnStdout: true).trim()
                    echo "▶ 선택된 브랜치: ${branch}"

                    def profile = 'local'
                    if (branch == 'main') {
                        profile = 'prod'
                    } else if (branch == 'develop') {
                        profile = 'dev'
                    } else if (branch == 'test') {
                        profile = 'test'
                    }

                    def tag = 'latest'

                    env.BRANCH = branch
                    env.ACTIVE_PROFILE = profile
                    env.IMAGE_TAG = tag
                    env.FULL_IMAGE_NAME = "${env.DOCKER_HUL_ID}/${env.IMAGE_NAME}:${env.IMAGE_TAG}"

                    echo "▶ 선택된 브랜치: ${env.BRANCH}"
                    echo "▶ 적용된 Spring Profile: ${env.ACTIVE_PROFILE}"
                    echo "▶ Docker 이미지 태그: ${env.IMAGE_TAG}"
                    echo "▶ Docker 전체 이미지 명: ${env.FULL_IMAGE_NAME}"
                }
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${env.FULL_IMAGE_NAME} ."
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'Docker-Hub_hulbo', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sh "echo $PASSWORD | docker login -u $USERNAME --password-stdin"
                    sh "docker push ${env.FULL_IMAGE_NAME}"
                }
            }
        }

        stage('Deploy to Remote Server') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'Docker-Hub_hulbo', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    script {
                        sh """
                            ssh -o StrictHostKeyChecking=no aws-service << 'EOF'
                            echo "$PASSWORD" | docker login -u "$USERNAME" --password-stdin
                            docker pull ${env.FULL_IMAGE_NAME}
                            docker stop ${env.IMAGE_NAME} || true
                            docker rm ${env.IMAGE_NAME} || true
                            docker run -d --name ${env.IMAGE_NAME} -p ${env.PORT} -e SPRING_PROFILES_ACTIVE=${env.ACTIVE_PROFILE} ${env.FULL_IMAGE_NAME}
                            EOF
                        """
                    }
                }
            }
        }

        stage('Set Spring Profile test') {
            steps {
                script {
                    echo "▶ 선택된 브랜치: ${env.BRANCH}"
                    echo "▶ 적용된 Spring Profile: ${env.ACTIVE_PROFILE}"
                    echo "▶ Docker 이미지 태그: ${env.IMAGE_TAG}"
                    echo "▶ Docker 전체 이미지 명: ${env.FULL_IMAGE_NAME}"
                }
            }
        }
    }
}

pipeline {
    agent any
    tools {
        maven "Maven_3.9.9"
    }

    environment {
        IMAGE_NAME = "hulbo/sc_discoverservice"
        IMAGE_TAG = ""
        ACTIVE_PROFILE = ""
    }

    stages {
        stage('Set Spring Profile') {
            steps {
                script {
                    // 브랜치명 기준 이미지 태그 설정 (ex: main -> latest, 나머지 -> 빌드번호)
                    def branch = env.BRANCH_NAME
                    env.IMAGE_TAG = (branch == 'main') ? 'latest' : env.BUILD_NUMBER

                    def profile = 'local'
                    if (branch == 'main') {
                        profile = 'prod'
                    } else if (branch == 'develop') {
                        profile = 'dev'
                    } else if (branch == 'test') {
                        profile = 'test'
                    }

                    env.ACTIVE_PROFILE = profile

                    echo "▶ 적용된 Spring Profile: ${env.ACTIVE_PROFILE}"
                    echo "▶ 선택된 브랜치: ${branch}"
                    echo "▶ Docker 이미지 태그: ${env.IMAGE_TAG}"
                }
            }
        }

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'Docker-Hub_hulbo', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sh "echo $PASSWORD | docker login -u $USERNAME --password-stdin"
                    sh "docker push ${IMAGE_NAME}:${IMAGE_TAG}"
                }
            }
        }

        stage('Deploy to Remote Server') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'Docker-Hub_hulbo', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    script {
                        def profile = env.ACTIVE_PROFILE
                        sh """
                            ssh -o StrictHostKeyChecking=no aws-service << 'EOF'
                            echo "$PASSWORD" | docker login -u "$USERNAME" --password-stdin
                            docker pull ${IMAGE_NAME}:${IMAGE_TAG}
                            docker stop sc_discoverservice || true
                            docker rm sc_discoverservice || true
                            docker run -d --name sc_discoverservice -p 8761:8761 -e SPRING_PROFILES_ACTIVE=${profile} ${IMAGE_NAME}:${IMAGE_TAG}
                            EOF
                        """
                    }
                }
            }
        }
    }
}

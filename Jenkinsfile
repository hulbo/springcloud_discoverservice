pipeline {
    agent any
    tools{
        maven "Maven_3.9.9"
    }
    parameters {
        string(name: 'BRANCH', defaultValue: 'main', description: 'Git 브랜치명 (예: main, develop, test 등)')
        booleanParam(name: 'USE_LATEST_TAG', defaultValue: true, description: '이미지 테그 latest 사용여부')
    }

    environment {
        IMAGE_NAME = "hulbo/sc_discoverservice"
        IMAGE_TAG = "${params.USE_LATEST_TAG ? 'latest' : env.BUILD_NUMBER}"
        ACTIVE_PROFILE = ""
    }
    stages {
        stage('Set Spring Profile') {
                steps {
                    script {
                    def branch = params.BRANCH
                    def profile = 'local'

                    if (branch == 'main') {
                        profile = 'prod'
                    } else if (branch == 'develop') {
                        profile = 'dev'
                    } else if (branch == 'test') {
                        profile = 'test'
                    }

                    // 값을 다음 스테이지에서도 쓰기 위해 env에 저장
                    env.ACTIVE_PROFILE = profile

                    echo "▶ 적용된 Spring Profile: ${env.ACTIVE_PROFILE}"
                    echo "▶ 선택된 브랜치: ${branch}"
                }
            }
        }

        stage('Checkout') {
            steps {
                git branch: "${params.BRANCH}",
                    credentialsId: 'Git-Hub_hulbo',
                    url: 'https://github.com/hulbo/springcloud_discoverservice'
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
                def profile = env.ACTIVE_PROFILE
                withCredentials([usernamePassword(credentialsId: 'Docker-Hub_hulbo', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ec2-user@43.201.25.43 << 'EOF'
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

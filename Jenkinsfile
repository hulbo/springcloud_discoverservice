pipeline {
    agent any
    tools{
        maven "Maven_3.9.9"
    }
    parameters {
        booleanParam(name: 'USE_LATEST_TAG', defaultValue: true, description: '이미지 테그 latest 사용여부')
        string(name: 'SPRING_PROFILE', defaultValue: 'prod', description: 'Spring active profile 설정')
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
                    def branch = env.BRANCH_NAME

                    if (branch == 'main') {
                        env.ACTIVE_PROFILE = 'prod'
                    } else if (branch == 'develop') {
                        env.ACTIVE_PROFILE = 'dev'
                    } else if (branch == 'test') {
                        env.ACTIVE_PROFILE = 'test'
                    } else {
                        env.ACTIVE_PROFILE = 'local'
                    }

                    echo "▶ 적용된 Spring Profile: ${env.ACTIVE_PROFILE}"
                }
            }
        }
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
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
                withCredentials([usernamePassword(credentialsId: 'Docker-Hub_hulbo', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ec2-user@43.201.25.43 << 'EOF'
                        echo "$PASSWORD" | docker login -u "$USERNAME" --password-stdin
                        docker pull ${IMAGE_NAME}:${IMAGE_TAG}
                        docker stop sc_discoverservice || true
                        docker rm sc_discoverservice || true
                        docker run -d --name sc_discoverservice -p 8761:8761 -e SPRING_PROFILES_ACTIVE=${ACTIVE_PROFILE} ${IMAGE_NAME}:${IMAGE_TAG}
                        EOF
                    """
                }
            }
        }
    }
}

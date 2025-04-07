pipeline {
    agent any
    tools {
        maven "Maven_3.9.9"
    }

    environment {
        IMAGE_NAME = "hulbo/sc_discoverservice"
        IMAGE_TAG = ""
        ACTIVE_PROFILE = ""
        BRANCH = "";
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
                    // git 명령어로 현재 브랜치 이름 추출
                    def branch = sh(script: "git rev-parse --abbrev-ref HEAD", returnStdout: true).trim()
                    env.IMAGE_TAG = (branch == 'main') ? 'latest' : env.BUILD_NUMBER
                    env.BRANCH = branch

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
                    echo "▶ 선택된 브랜치: ${env.BRANCH}"
                    echo "▶ Docker 이미지 태그: ${env.IMAGE_TAG}"
                }
            }
        }
    }
}

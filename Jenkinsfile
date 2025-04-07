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
                    def branch = sh(script: "git branch -r --contains HEAD | grep origin/ | head -n 1 | sed 's@origin/@@'", returnStdout: true).trim()
                    echo "▶ 선택된 브랜치: ${branch}"

                    if (branch == 'main') {
                        profile = 'prod'
                    } else if (branch == 'develop') {
                        profile = 'dev'
                    } else if (branch == 'test') {
                        profile = 'test'
                    }

                    // 운영은 빌드 넘버로 진행해서 백업 하게 변경
                    // def tag = (branch == 'main') ? "${env.BUILD_NUMBER}" : 'latest'
                    def tag = 'latest'

                    // withEnv로 다음 스테이지에서도 사용할 수 있도록 환경 변수 설정
                    withEnv([
                        "BRANCH=${branch}",
                        "ACTIVE_PROFILE=${profile}",
                        "IMAGE_TAG=${tag}"
                    ]) {
                        echo "▶ 선택된 브랜치: ${env.BRANCH}"
                        echo "▶ 적용된 Spring Profile: ${env.ACTIVE_PROFILE}"
                        echo "▶ Docker 이미지 태그: ${env.IMAGE_TAG}"
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
                }
            }
        }
    }
}

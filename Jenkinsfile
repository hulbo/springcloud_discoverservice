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

                    def profile = 'local'
                                        if (branch == 'main') {
                                            profile = 'prod'
                                        } else if (branch == 'develop') {
                                            profile = 'dev'
                                        } else if (branch == 'test') {
                                            profile = 'test'
                                        }

                                        env.ACTIVE_PROFILE = profile
                                        env.BRANCH = branch

                                        echo "▶ 적용된 Spring Profile: ${env.ACTIVE_PROFILE}"
                                        echo "▶ 선택된 브랜치: ${env.BRANCH}"
                                        echo "▶ Docker 이미지 태그: ${env.IMAGE_TAG}"
                }
            }
        }
    }
}

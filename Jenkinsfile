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
                }
            }
        }
    }
}

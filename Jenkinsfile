pipeline {
    agent any
    tools{
        maven "Maven_3.9.9"
    }
    environment {
        IMAGE_NAME = "hulbo/sc_discoverservice"
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
                sh "docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} ."
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'Docker-Hub_hulbo', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sh "echo $PASSWORD | docker login -u $USERNAME --password-stdin"
                    sh "docker push ${IMAGE_NAME}:${BUILD_NUMBER}"
                }
            }
        }
    }
}

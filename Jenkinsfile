pipeline {
    agent any

    environment {
        GIT_CREDENTIALS_ID = 'github-creds'
        DOCKER_CREDENTIALS_ID = 'dockerhub-creds'
        DOCKER_IMAGE = 'tharindu1996/test-springboot'
    }

    tools {
        maven 'Maven 3.8.5'
        jdk 'JDK17'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[
                        url: 'https://github.com/tharinduVibudda/spring-petclinic-test.git',
                        credentialsId: "${env.GIT_CREDENTIALS_ID}"
                    ]]
                ])
            }
        }

        // stage('Build & Test') {
        //     steps {
        //         sh 'mvn clean test -Dspring.docker.compose.skip=true -Dspring.profiles.active=""'
                
        //     }
        // }

        stage('Unit Test') {
            steps {
                 sh 'mvn test -Dspring.docker.compose.skip=true -Dspring.profiles.active=""'
                
            }
        }

        stage('Package') {
            steps {
                sh 'mvn package -DskipTests'
                
            }
        }
        

        stage('Archive JAR') {
            steps {
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${env.DOCKER_IMAGE}:${env.BUILD_NUMBER}")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', "${env.DOCKER_CREDENTIALS_ID}") {
                        docker.image("${env.DOCKER_IMAGE}:${env.BUILD_NUMBER}").push()
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Build and deployment successful!'
        }
        failure {
            echo 'Build failed!'
        }
    }
}

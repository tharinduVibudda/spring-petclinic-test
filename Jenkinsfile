pipeline {
    agent any

    environment {
        GIT_CREDENTIALS_ID = 'github-creds'
        DOCKER_CREDENTIALS_ID = 'dockerhub-creds'
        DOCKER_IMAGE = 'tharindu1996/test-springboot'
        AZURE_DEPLOY_CREDENTIALS_ID = 'azure-deploy-creds'
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

        stage('Build & Test') {
            steps {
                sh 'mvn clean test -Dtest="!*PostgresIntegrationTests"'
                
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

        stage('Deploy to Azure App Service') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${env.AZURE_DEPLOY_CREDENTIALS_ID}", usernameVariable: 'AZUSER', passwordVariable: 'AZPASS')]) {
                    sh '''
                        cd target
                        mkdir deploy
                        cp *.jar deploy/app.jar
                        cd deploy
                        git init
                        git config user.email "ci@jenkins"
                        git config user.name "Jenkins CI"
                        git add .
                        git commit -m "Deploy build to Azure App Service"
                        git remote add azure https://${AZUSER}:${AZPASS}@spring-petclinic-app.scm.azurewebsites.net/spring-petclinic-app.git
                        git push --force azure master
                    '''
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

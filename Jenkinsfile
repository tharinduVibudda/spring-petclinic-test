pipeline {
    agent any

    environment {
        GIT_CREDENTIALS_ID = 'github-creds'
        DOCKER_CREDENTIALS_ID = 'dockerhub-creds'
        DOCKER_IMAGE = 'tharindu1996/test-springboot'
        AZURE_SP_CREDENTIALS_ID = 'azure-sp-json' // this is the new secret text credential
        RESOURCE_GROUP = 'petclinic-rg'
        APP_NAME = 'spring-petclinic-app'
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
       stage('Deploy to Azure App Service (ZIP)') {
            steps {
                withCredentials([string(credentialsId: "${env.AZURE_SP_CREDENTIALS_ID}", variable: 'AZURE_CREDENTIALS_JSON')]) {
                    sh '''
                        echo "$AZURE_CREDENTIALS_JSON" > azureauth.json
                        
                        # Log in using Service Principal
                        az login --service-principal \
                          --username $(jq -r .clientId azureauth.json) \
                          --password $(jq -r .clientSecret azureauth.json) \
                          --tenant $(jq -r .tenantId azureauth.json)

                        # Prepare ZIP for deployment
                        cd target
                        mkdir deploy
                        cp *.jar deploy/app.jar
                        cd deploy
                        zip -r ../../deploy.zip .
                        cd ../..

                        # Deploy to Azure App Service
                        az webapp deployment source config-zip \
                          --resource-group "$RESOURCE_GROUP" \
                          --name "$APP_NAME" \
                          --src deploy.zip
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

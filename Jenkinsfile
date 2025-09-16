pipeline {
    agent any
    environment {
        APP_NAME = 'petclinic-app'
        NEXUS_URL = 'localhost:8083/repository/docker-hosted'
        NEXUS_CREDENTIALS_ID = 'nexus-credentials'
    }

    stages {
        stage('Checking environment variables') {
            steps {
                echo "BRANCH_NAME  = ${env.BRANCH_NAME}"
                echo "BUILD_NUMBER = ${env.BUILD_NUMBER}"
            }
        }
        stage('Checkout') {
            steps {
                echo "Cloning Repository"
                checkout scm
            }
        }
        stage('Build Project') {
            steps {
                echo "Cleaning and Packaging Project"
                sh './mvnw clean package -DskipTests'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                echo "Running SonarQube analysis"
                withSonarQubeEnv('SonarQube-server') {
                    withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                        sh """
                          ./mvnw sonar:sonar \
                          -Dsonar.projectKey=${APP_NAME} \
                          -Dsonar.login=$SONAR_TOKEN
                        """
                    }
                }
            }
        }
        stage('Check Credentials') {
            steps {
                script {
                    echo "Listing available credentials..."
                    def creds = com.cloudbees.plugins.credentials.CredentialsProvider.lookupCredentials(
                        com.cloudbees.plugins.credentials.common.StandardUsernamePasswordCredentials.class,
                        Jenkins.instance,
                        null,
                        null
                    )
                    creds.each { println("ID: ${it.id}, Username: ${it.username}") }
                }
            }
        }
        stage('Build and Push Docker Image') {
            steps {
                script {
                    def fullImageTag = "${NEXUS_URL}/${APP_NAME}:${env.BUILD_NUMBER}"
                    docker.withRegistry("http://${NEXUS_URL}", "nexus-credentials") {
                        echo "Building Docker image: ${fullImageTag}"
                        def customImage = docker.build(fullImageTag)
                        echo "Pushing Docker image to Nexus..."
                        customImage.push()
                        echo "Image pushed successfully!"
                    }
                }
            }
        }
    }
    post {
        success {
            archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
        }
        always {
            echo "Pipeline finished."
            cleanWs()
        }
    }
}
//test1
//test2
//test3
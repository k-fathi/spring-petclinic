pipeline {
    agent any

    environment {
        APP_NAME = 'petclinic-app'   
        NEXUS_HOSTED_URL = 'localhost:8083'
        NEXUS_CREDENTIALS_ID = 'nexus-credentials'
    }

    stages {
        stage('Checking environment varaibles'){
            steps {
                echo "BRANCH_NAME  = ${env.BRANCH_NAME}"
                echo "BUILD_NUMBER = ${env.BUILD_NUMBER}"
            }
        }     
        stage('checkout'){
            steps {
                echo "Clonning Reposotiory"
                checkout scm
            }
        }  
        stage('Cleaning and Packaging') {
            steps {
                echo "Cleaning and Packaging Project"
                sh './mvnw clean'
            }
        }      
        stage('SonarQube Analysis') {
            steps {
                echo "Running SonarQube analysis"
                withSonarQubeEnv('SonarQube-server') {
                    withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                        sh './mvnw sonar:sonar -Dsonar.login=$SONAR_TOKEN'
                    }
                }
            }
        }
        stage('Build and Push to Nexus') {
            steps {
                script {
                    def fullImageTag = "${NEXUS_HOSTED_URL}/${APP_NAME}:${env.BUILD_NUMBER}"
                    docker.withRegistry("http://${NEXUS_HOSTED_URL}", NEXUS_CREDENTIALS_ID) {
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
        always {
            echo "Pipeline finished."
            cleanWs()
        }
    }
}
//test1
//test2
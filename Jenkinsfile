pipeline {
    agent any

    tools {
        jdk 'JDK21'
        maven 'Maven-3.9'
    }

    environment {
        SCANNER_HOME = tool 'SonarScanner'

        DOCKER_USER = "hemanthpoojary"

        BACKEND_IMAGE = "hemanthpoojary/smartims-backend:v1"
        FRONTEND_IMAGE = "hemanthpoojary/smartims-frontend:v1"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Backend') {
            steps {
                dir('backend') {
                    sh 'mvn clean package -DskipTests'
                }
            }
        }

        stage('Build Frontend') {
            steps {
                dir('frontend') {
                    sh '''
                    export PATH=/usr/local/bin:$PATH
                node -v
                npm -v
                npm install
                npm run build
            
                    '''
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {

                    sh """
                    ${SCANNER_HOME}/bin/sonar-scanner \
                    -Dsonar.projectKey=smart-incident-management \
                    -Dsonar.projectName=smart-incident-management \
                    -Dsonar.sources=. \
                    -Dsonar.java.binaries=backend/target
                    """

                }
            }
        }

/*
        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

*/  
      stage('Build Backend Docker Image') {
            steps {
                dir('backend') {
                    sh "docker build -t ${BACKEND_IMAGE} ."
                }
            }
        }

        stage('Build Frontend Docker Image') {
            steps {
                dir('frontend') {
                    sh "docker build -t ${FRONTEND_IMAGE} ."
                }
            }
        }

        stage('DockerHub Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub',
                    usernameVariable: 'USERNAME',
                    passwordVariable: 'PASSWORD'
                )]) {

                    sh '''
                    echo $PASSWORD | docker login -u $USERNAME --password-stdin
                    '''

                }
            }
        }

        stage('Push Backend Image') {
            steps {
                sh "docker push ${BACKEND_IMAGE}"
            }
        }

        stage('Push Frontend Image') {
            steps {
                sh "docker push ${FRONTEND_IMAGE}"
            }
        }

        stage('Deploy') {
            steps {

                sh '''
                docker stop smartims-backend || true
                docker rm smartims-backend || true

                docker stop smartims-frontend || true
                docker rm smartims-frontend || true
                '''

                sh """
                docker run -d \
                --name smartims-backend \
                -p 8080:8080 \
                ${BACKEND_IMAGE}
                """

                sh """
                docker run -d \
                --name smartims-frontend \
                -p 3000:80 \
                ${FRONTEND_IMAGE}
                """

            }
        }

    }

    post {

        success {
            echo "Pipeline completed successfully."
        }

        failure {
            echo "Pipeline failed."
        }

        always {
            sh 'docker image prune -f'
        }

    }

}

pipeline {
    agent { label "dev" }
    
    tools {
        jdk 'jdk17'
        maven 'maven'
    }
    
    environment {
        SONAR_HOST_URL = 'http://13.234.186.141:9000/'
        SONAR_PROJECT_KEY = 'two-tier-flask-app'
        SONAR_PROJECT_NAME = 'Two-Tier Flask App'
    }
    
    stages {
        stage("Code Clone") {
            steps {
                git url: "https://github.com/julkar09/two-tier-flask-app.git", branch: "test"
            }
        }

        stage("Sonar Analysis") {
            steps {
                withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_LOGIN')]) {
                    sh """
                        mvn clean package
                        mvn sonar:sonar \
                            -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                            -Dsonar.projectName="${SONAR_PROJECT_NAME}" \
                            -Dsonar.java.binaries=target/classes \
                            -Dsonar.host.url=${SONAR_HOST_URL} \
                            -Dsonar.login=${SONAR_LOGIN}
                    """
                }
            }
        }

        stage("Build") {
            steps {
                sh "docker build -t flask-app:${BUILD_NUMBER} ."
            }
        }

        stage("Trivy File System Scan") {
            steps {
                sh "trivy fs ."
            }
        }

        stage("Docker Scout Analysis") {
            steps {
                sh "docker scout quickview flask-app:${BUILD_NUMBER}"
            }
        }

        stage("Push to Docker Hub") {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: "dockerHubCreds",
                    usernameVariable: "DOCKER_HUB_USER",
                    passwordVariable: "DOCKER_HUB_PASS"
                )]) {
                    sh """
                        docker login -u ${DOCKER_HUB_USER} -p ${DOCKER_HUB_PASS}
                        docker tag flask-app:${BUILD_NUMBER} ${DOCKER_HUB_USER}/flask-app:${BUILD_NUMBER}
                        docker push ${DOCKER_HUB_USER}/flask-app:${BUILD_NUMBER}
                    """
                }
            }
        }

        stage("Deploy") {
            steps {
                sh "docker compose up -d"
            }
        }
    }

    post {
        success {
            emailext(
                to: 'zulkarnineador7@gmail.com',
                subject: 'Build Successful!',
                body: 'Good news: Your build was successful!'
            )
        }
        failure {
            emailext(
                to: 'zulkarnineador7@gmail.com',
                subject: 'Build Failed',
                body: 'Bad news: Your build failed.'
            )
        }
    }
}

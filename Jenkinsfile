pipeline {
    agent { label "dev" }
    tools {
        jdk 'jdk17'
        maven 'maven'
    }
    environment {
        SONAR_HOST_URL = 'http://15.207.85.64:9000/'
        SONAR_PROJECT_KEY = 'two-tier-flask-app'
        SONAR_PROJECT_NAME = 'Two-Tier Flask App'
        SONAR_LOGIN = 'squ_e947028e658a604cd5be88fa2026d4a22e52bde5'
    }
    stages {
        stage("Code Clone") {
            steps {
                script {
                    git url: "https://github.com/julkar09/two-tier-flask-app.git", branch: "test"
                }
            }
        }
        stage("Sonar Analysis") {
            steps {
                sh "mvn clean package"
                sh """ 
                    mvn sonar:sonar \
                        -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                        -Dsonar.projectName="${SONAR_PROJECT_NAME}" \
                        -Dsonar.java.binaries=target/classes \
                        -Dsonar.host.url=${SONAR_HOST_URL} \
                        -Dsonar.login=${SONAR_LOGIN}
                """
            }
        }
        stage("Build") {
            steps {
                sh "docker build -t flask-app:0 ."
            }
        }
        stage("Trivy File System Scan") {
            steps {
                sh "trivy fs ."
            }
        }
        stage("Trivy Image Scan") {
            steps {
                sh "trivy image --severity HIGH,CRITICAL flask-app:0"
            }
        }
        stage("Push to Docker Hub") {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: "dockerHubCreds",
                    passwordVariable: "dockerHubPass",
                    usernameVariable: "dockerHubUser"
                )]) {
                    sh """
                        echo ${env.dockerHubPass} | docker login -u ${env.dockerHubUser} --password-stdin
                        docker image tag flask-app:0 ${env.dockerHubUser}/flask-app:0
                        docker push ${env.dockerHubUser}/flask-app:0
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
            emailext body: 'Good news: Your build was successful!',
                     subject: 'Build Successful!',
                     to: 'zulkarnineador7@gmail.com'
        }
        failure {
            emailext body: 'Bad news: Your build failed.',
                     subject: 'Build Failed',
                     to: 'zulkarnineador7@gmail.com'
        }
    }
}

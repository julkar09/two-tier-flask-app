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
        SONAR_LOGIN = 'squ_bd5f7af0f00d43410d52f1ef80fba3c4754b7cc9'
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
                sh ''' 
                    mvn sonar:sonar \
                        -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                        -Dsonar.projectName="${SONAR_PROJECT_NAME}" \
                        -Dsonar.java.binaries=target/classes \
                        -Dsonar.host.url=${SONAR_HOST_URL} \
                        -Dsonar.login=${SONAR_LOGIN}
                '''
            }
        }
        stage("Build") {
            steps {
                sh "docker build -t flask-app:0 ."
            }
        }
        stage("Trivy File System Scan") {
            steps {
                script {
                    sh "trivy fs ."
                }
            }
        }
        stage("Docker Scout Analysis") {
            steps {
                script {
                    sh "docker scout quickview flask-app:0"
                    sh "docker scout cves flask-app:0"
                }
            }
        }
        stage("Push to Docker Hub") {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: "dockerHubCreds",
                    passwordVariable: "dockerHubPass",
                    usernameVariable: "dockerHubUser"
                )]) {
                    sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPass}"
                    sh "docker image tag flask-app:0 ${env.dockerHubUser}/flask-app:0"
                    sh "docker push ${env.dockerHubUser}/flask-app:0"
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

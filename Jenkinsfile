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
                withSonarQubeEnv('SonarQube') {  // Use the SonarQube server configured in Jenkins
                    sh "mvn clean package"
                    sh ''' 
                        mvn sonar:sonar \
                            -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                            -Dsonar.projectName="${SONAR_PROJECT_NAME}" \
                            -Dsonar.java.binaries=target/classes \
                            -Dsonar.host.url=${SONAR_HOST_URL} \
                            -Dsonar.login=$SONARQUBE_TOKEN
                    '''
                }
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

        stage("Docker Scout Analysis") {
            steps {
                sh "docker scout quickview flask-app:0"
            }
        }

        stage("Push to Docker Hub") {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: "dockerHubCreds",
                    passwordVariable: "dockerHubPass",
                    usernameVariable: "dockerHubUser"
                )]) {
                    sh "docker login -u ${dockerHubUser} -p ${dockerHubPass}"
                    sh "docker tag flask-app:0 ${dockerHubUser}/flask-app:0"
                    sh "docker push ${dockerHubUser}/flask-app:0"
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

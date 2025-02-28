pipeline{
    agent { label "dev"};

    stages {
        stage("Code Clone") {
            steps {
                script {
                    git url: "https://github.com/julkar09/two-tier-flask-app.git", branch: "test"
                }
            }
        }
        stage("Build") {
            steps {
                sh "docker build -t flask-app:0 ."
            }
        }
		  stage("Trivy File System Scan"){
            steps{
                script{
                    trivy_fs()
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
        emailext body: 'good news: your build was successful!',
                 subject : 'Build successful!',
                 to: 'zulkarnineador7@gmail.com'
        
    }
    failure {
        emailext body: 'bad news: your build was fail',
                 subject : 'Build failed',
                 to: 'zulkarnineador7@gmail.com'     
        
    }
    
}
}

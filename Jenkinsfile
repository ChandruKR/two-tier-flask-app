pipeline {
            agent { label 'TEst' }

    stages {
        stage('Github Code Stage') {
            steps {
                git url: "https://github.com/ChandruKR/two-tier-flask-app.git", branch: 'master'
                script {
                    def output = sh(returnStdout: true, script: 'uname -r')
                    echo "Output: ${output}"
                }
            }
        }
        stage('Build Stage') {
            steps {
                sh "docker build -t local_flask-app:latest ."

            }
        }
        stage('Dockerhub tag & push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'Dockerhubid',
                passwordVariable: 'dockerHubPassword',
                usernameVariable: 'dockerHubUser')]) {
                    sh "docker image tag  local_flask-app:latest   ${env.dockerHubUser}/flask-app:latest"
                    sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPassword}"
                    sh "docker push ${env.dockerHubUser}/flask-app:latest"

                }

            }
        }
        stage('Deploy Under Agent "TEst"') {
            steps {
                sh "docker-compose up -d "
            }
        }
    }
post {
    success {
        emailext body: "Build Successfull",
            subject: "Build Successfull",
            to: "ckfordevops0114@gmail.com"

    }
    failure {
        emailext body: "Build Failed",
            subject: "Build Failed",
            to: "ckfordevops0114@gmail.com"

    }
}
}

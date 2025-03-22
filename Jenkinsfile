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

        
        stage('Trivy file system scan') {
            steps {
                sh "trivy fs . -o results.json"

            }
        }
                stage('Build Stage') {
            steps {
                sh "docker build -t local_flask-app:latest ."

            }
        }
        
                stage('Trivy Image Scan') {
            steps {
            script {
                def scanOutput = sh(returnStatus: true, script: 'trivy image --exit-code 1 --severity CRITICAL local_flask-app:latest')
                if (scanOutput != 0) {
                    error("Critical vulnerabilities found in the image. Failing the build.")
                }
            }
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
        emailext body: "The build has completed successfully. Check details at: ${env.BUILD_URL}", attachLog: true,
            subject: "Build Notification: ${env.JOB_NAME} - Build #${env.BUILD_NUMBER}",
            to: "ckfordevops0114@gmail.com"

    }
    failure {
        emailext body: "Build Failed. Check details at: ${env.BUILD_URL} WORKSPACE: ${WORKSPACE}", attachLog: true,
        from: "noreplychandru@gmail.com",
        subject: "Build Notification: ${env.JOB_NAME} - Build #${env.BUILD_NUMBER}",
        recipientProviders: [[$class: 'DevelopersRecipientProvider']],
        to: "ckfordevops0114@gmail.com"
    }
}
}

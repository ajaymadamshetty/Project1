pipeline {
    agent any

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/ajaymadamshetty/Project1/'
            }
        }
        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }
        stage('Grype FS Scan') {
            steps {
                sh "grype dir:. --output table > fs.html"
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh "mvn sonar:sonar"
                }
            }
        }
        stage('Build') {
            steps {
                sh "mvn package"
            }
        }
        stage('Publish Artifacts') {
            steps {
                sh "mvn deploy"
            }
        }
        stage('Docker Build & Tag') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'dockerhub-cred', url: 'https://index.docker.io/v1/') {
                        sh "docker build -t ugogabriel/gab-blogging-app ."
                    }
                }
            }
        }
        stage('Grype Image Scan') {
            steps {
                sh "grype ugogabriel/gab-blogging-app:latest --output table > image.html"
            }
        }
        stage('Docker Push Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'dockerhub-cred', url: 'https://index.docker.io/v1/') {
                        sh "docker push ugogabriel/gab-blogging-app"
                    }
                }
            }
        }
        stage('K8s Deploy') {
            steps {
                withKubeCredentials(kubectlCredentials: [[caCertificate: '', clusterName: 'devopsshack-cluster', contextName: '', credentialsId: 'k8s-token', namespace: 'webapps', serverUrl: 'https://AD1D9143EC6B3C8A72B36759FA28854D.gr7.eu-west-2.eks.amazonaws.com']]) {
                    sh "kubectl apply -f deployment-service.yml"
                    sleep 20
                }
            }
        }
        stage('Verify Deployment') {
            steps {
                withKubeCredentials(kubectlCredentials: [[caCertificate: '', clusterName: 'devopsshack-cluster', contextName: '', credentialsId: 'k8s-token', namespace: 'webapps', serverUrl: 'https://AD1D9143EC6B3C8A72B36759FA28854D.gr7.eu-west-2.eks.amazonaws.com']]) {
                    sh "kubectl get pods"
                    sh "kubectl get service"
                }
            }
        }
    }
}

post {
    always {
        script {
            def jobName = env.JOB_NAME
            def buildNumber = env.BUILD_NUMBER
            def pipelineStatus = currentBuild.result ?: 'UNKNOWN'
            pipelineStatus = pipelineStatus.toUpperCase()
            def bannerColor = pipelineStatus == 'SUCCESS' ? 'green' : 'red'

            def body = """
            <body>
                <div style="border: 2px solid ${bannerColor}; padding: 10px;">
                    <h3 style="color: ${bannerColor};">
                        Pipeline Status: ${pipelineStatus}
                    </h3>
                    <p>Job: ${jobName}</p>
                    <p>Build Number: ${buildNumber}</p>
                    <p>Status: ${pipelineStatus}</p>
                </div>
            </body>
            """

            emailext(
                subject: "${jobName} - Build ${buildNumber} - ${pipelineStatus}",
                body: body,
                to: 'madamshettyajay@gmail.com',
                from: 'jenkins@example.com',
                replyTo: 'jenkins@example.com',
                mimeType: 'text/html'
            )
        }
    }
}

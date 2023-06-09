pipeline { 
    agent any

    environment {
        registry = "sen31088"  // Replace with your Docker registry URL
        DOCKERHUB_CREDENTIALS= credentials('docker-sen31088')
        imageName = "webapp"  // Replace with your desired image name
        containerName = "my-webapp-container"  // Replace with your desired container name
        dockerfilePath = "./Dockerfile"  // Replace with the path to your Dockerfile
        dockerArgs = "-p 8080:80"  // Replace with your desired container arguments
        version = sh(script: 'jq \'.version\' version.json', returnStdout: true).trim()
    }
    stages {
        stage('Clean WS') { 
            steps { 
                cleanWs()
            }
        }
        stage('SCM'){
            steps {
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'sen', url: 'https://github.com/sen31088/app1.git']])
            }
        }
        
        stage('Docker Login') {
            steps {
                   sh 'echo $DOCKERHUB_CREDENTIALS_PSW | sudo docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'     		 
                }
            }
        

        stage('Build') {
            steps {
                
                sh "sudo docker build -t ${registry}/${imageName}:${version} -f ${dockerfilePath} ."
            }
        }
        
        stage('Push to Artifcatory') {
            steps {
                sh "sudo docker push ${registry}/${imageName}:${version}"
            }
        }


        stage('Deploy') {
            steps {
                sh "ssh root@10.0.1.207 'docker rm -f ${containerName} || true'"

                sh "ssh root@10.0.1.207 'docker run -d --name ${containerName} ${dockerArgs} ${registry}/${imageName}:${version}' "
            }
        }
    }
}

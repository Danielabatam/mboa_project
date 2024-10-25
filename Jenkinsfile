pipeline {
    agent any
    parameters {
        string(name: 'github-url', defaultValue: '', description: 'Enter your GitHub URL')
        string(name: 'image-name', defaultValue: 'dockerhubusername/repo-name', description: 'Enter your image name')
        string(name: 'image-tag', defaultValue: 'latest', description: 'Enter your image tag')  
    }

    stages {
        stage("Clone repository") {
            steps {
                git branch: 'main', url: "${params['github-url']}", credentialsId: "github_daniela"
            }
        }
        stage("Build Dockerfile") {
            steps {
                script {
                    sh "docker build -t ${params['image-name']}:${params['image-tag']} ."
                }
            }
        }
        stage("Connect to DockerHub") {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: "dockerHub_daniela", 
                    usernameVariable: "dockerusername", passwordVariable: "dockerhubpassword")]) {
                        sh "docker login -u $dockerusername -p $dockerhubpassword"
                    }
                }
            }
        }
        stage("Push to DockerHub") {
            steps {
                script {
                    sh "docker push ${params['image-name']}:${params['image-tag']}"
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}

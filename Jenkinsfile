pipeline {
    agent any
    
    parameters {
        string(name: 'github-url', defaultValue: '', description: 'Enter your GitHub URL')
        string(name: 'image-name', defaultValue: 'dockerhubusername/repo-name', description: 'Enter your image name')
        string(name: 'image-tag', defaultValue: 'latest', description: 'Enter your image tag')  
        boolean(name: 'skip-stage', defaultValue: 'false', description: "Mark for yes, leave empty for false")  
    }
    environment {
        scanner = tool 'sonar'
    }

    stages {
        stage("Clone repository") {
            steps {
                git branch: 'main', url: "${params['github-url']}", credentialsId: "github_daniela"
            }
        }
        stage("Code scan") {
            when {
                expression { !params.skip }
            }
         {
            steps {
                withCredentials([usernamePassword(credentialsId: "sonar", usernameVariable: 'SONAR_TOKEN')]) {
                    script {
                        withSonarQubeEnv('sonar') {
                            sh '''
                                $scanner/bin/sonar-scanner \
                                -Dsonar.login=$SONAR_TOKEN \
                                -Dsonar.host.url=http://18.216.0.51:9000/ \
                                -Dsonar.projectKey=inance_daniela \
                                -Dsonar.sources=./inance_daniela
                            '''
                        }
                    }
                }
            }
        }
        stage("Build Dockerfile") {
            when {
                expression { !params.skip }
            }
            steps {
                script {
                    sh "docker build -t ${params['image-name']}:${params['image-tag']} ."
                }
            }
        }
        stage("Connect to DockerHub") {
            when {
                expression { !params.skip }
            }
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
            when {
                expression { !params.skip-stage }
            }
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

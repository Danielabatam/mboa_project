pipeline {
    agent any
    parameters {
        string(name: 'github-url', defaultValue: '', description: 'Enter your GitHub URL')
        string(name: 'image-name', defaultValue: 'dockerhubusername/repo-name', description: 'Enter your image name')
        string(name: 'image-tag', defaultValue: 'latest', description: 'Enter your image tag')  
        boolean(name: 'skip-stage', defaultValue: 'false', description: "mark for yes leave empty for false")  
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
            withCredentials([usernamePassword(credentialsId: "sonar", variables: 'SONAR_TOKEN')])
            steps {
                script {
                    withsonarqubeEnv('sonar') {
                        sh '''
                        $scanner/bin/sonnar-scanner \
                        -Dsonnar.login=sonar \
                        -Dsonnar.host.url=http://18.216.0.51:9000/ \
                        -Dsonnar.projectKet=inance_daniela \
                        -Dsonnar.source=./inance_daniela
                        '''
                        
                    }
                }
            }
        }
        stage("Build Dockerfile") {
            when {
                expression {params.skip-stage}
            }
            steps {
                script {
                    sh "docker build -t ${params['image-name']}:${params['image-tag']} ."
                }
            }
        }
        stage("Connect to DockerHub") {
            when {
                expression {params.skip-stage}
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
                expression {params.skip-stage}
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

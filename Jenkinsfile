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
            // when {
            //     expression { !params.skip-stage } // Exécute cette étape si skip-stage est faux
            // }
            steps {
                git branch: 'main', url: "${params['github-url']}", credentialsId: "github_daniela"
            }
        }
        
        stage("Code scan") {
            when {
                expression { !params.skip-stage } // Exécute cette étape si skip-stage est faux
            }
            steps {
                withCredentials([usernamePassword(credentialsId: "sonar", usernameVariable: 'SONAR_TOKEN')]) {
                    script {
                        withSonarQubeEnv('sonar') {
                            sh '''
                                $scanner/bin/sonar-scanner \
                                -Dsonar.login=$SONAR_TOKEN \
                                -Dsonar.host.url=http://3.17.190.122:9000/ \
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
                expression { !params.skip-stage } // Exécute cette étape si skip-stage est faux
            }
            steps {
                script {
                    sh "docker build -t ${params['image-name']}:${params['image-tag']} ."
                }
            }
        }
        
        stage("Connect to DockerHub") {
            when {
                expression { !params.skip-stage } // Exécute cette étape si skip-stage est faux
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
                expression { !params.skip-stage } // Exécute cette étape si skip-stage est faux
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
            cleanWs() // Nettoie l'espace de travail après l'exécution du pipeline
        }
    }
}

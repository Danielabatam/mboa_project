pipeline {
    agent any // Utilise n'importe quel agent disponible pour exécuter le pipeline
    parameters {
        // Paramètres pour spécifier l'URL GitHub, le nom et le tag de l'image Docker, et une option pour sauter certaines étapes
        string(name: 'github-url', defaultValue: '', description: 'Enter your GitHub URL')
        string(name: 'image-name', defaultValue: 'dockerhubusername/repo-name', description: 'Enter your image name')
        string(name: 'image-tag', defaultValue: 'latest', description: 'Enter your image tag')  
        boolean(name: 'skip-stage', defaultValue: false, description: "Mark for yes, leave empty for false")  
    }
    environment {
        scanner = tool 'sonar' // Définit l'outil SonarScanner pour l'analyse de code
    }

    stages {
        // Étape 1 : Cloner le dépôt depuis GitHub
        stage("Clone repository") {
            steps {
                git branch: 'main', url: "${params['github-url']}", credentialsId: "github_daniela"
                // Clone la branche principale depuis l'URL GitHub spécifiée, en utilisant des identifiants sécurisés
            }
        }

        // Étape 2 : Analyser le code avec SonarQube
        stage("Code scan") {
            steps { // Récupère les informations d'identification pour SonarQube
                withCredentials([usernamePassword(credentialsId: "sonar", usernameVariable: 'SONAR_TOKEN')]) {
                    script {  // Configure SonarQube pour analyser le code source
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

        // Étape 3 : Construire l'image Docker à partir du Dockerfile
        stage("Build Dockerfile") {
            when {
                expression { params.skip-stage } // Exécute cette étape seulement si skip-stage est activé
            }
            steps {
                script {
                    sh "docker build -t ${params['image-name']}:${params['image-tag']} ." // Construit l'image Docker avec le nom et le tag spécifiés
                }
            }
        }

        // Étape 4 : Se connecter à DockerHub
        stage("Connect to DockerHub") {
            when {
                expression { params.skip-stage } // Exécute cette étape seulement si skip-stage est activé
            }
            steps {
                script {
                    // Utilise les informations d'identification pour se connecter à DockerHub
                    withCredentials([usernamePassword(credentialsId: "dockerHub_daniela", 
                    usernameVariable: "dockerusername", passwordVariable: "dockerhubpassword")]) {
                        sh "docker login -u $dockerusername -p $dockerhubpassword"
                    }
                }
            }
        }

        // Étape 5 : Pousser l'image Docker vers DockerHub
        stage("Push to DockerHub") {
            when {
                expression { params.skip-stage } // Exécute cette étape seulement si skip-stage est activé
            }
            steps {
                script {
                    sh "docker push ${params['image-name']}:${params['image-tag']}"
                    // Pousse l'image Docker vers le dépôt DockerHub
                }
            }
        }
    }

    post {
        // Actions à effectuer après l'exécution du pipeline
        always {
            cleanWs() // Nettoie l'espace de travail après la fin du pipeline
        }
    }
}

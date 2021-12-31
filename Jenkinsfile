pipeline {
    agent any

    stages {

        stage("Initial cleanup") {
            steps {
                dir("${WORKSPACE}") {
                deleteDir()
                }
            }
        }
  
        stage('Checkout SCM') {
            steps {
                git branch: 'main', url: 'https://github.com/Anefu/php-todo.git'
            }
        }
        
        stage('Build image') {
            steps {
                sh "docker build -t anefu/php-todo:${env.BRANCH_NAME}-${env.BUILD_NUMBER} ."
            }
        }
        stage("Start the app") {
            steps {
                sh "docker-compose up -d"
            }
        }
        stage("Test endpoint") {
            steps {
                script {
                    while (true) {
                        def response = httpRequest 'http://localhost:8000'
                        if (response.status == 200) {
                            withCredentials([usernamePassword(credentialsId: 'dockerHub', passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUser')]) {
                                sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPassword}"
                                sh "docker push anefu/php-todo:${env.BRANCH_NAME}-${env.BUILD_NUMBER}"
                            }
                            break 
                        }
                    }
                }
            }
        }
        // stage('Docker Push') {
        //     when { expression { response.status == 200 } }
        //     steps {
        //         withCredentials([usernamePassword(credentialsId: 'dockerHub', passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUser')]) {
        //             sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPassword}"
        //             sh "docker push anefu/php-todo:${env.BRANCH_NAME}-${env.BUILD_NUMBER}"
        //         }
        //     }
        // }
        stage ("Remove images") {
            steps {
                sh "docker system prune -af"
            }
        }
    }
}

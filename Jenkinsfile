pipeline {
    agent any
    stages {
        stage ('Compile Stage') {

            steps {
                withMaven(maven : 'maven') {
                    sh 'mvn clean compile'
                }
            }
        }
        
        stage ('Build') {
            steps {
                withMaven(maven : 'maven') {
                    sh 'mvn package'
                }
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    app = docker.build("pushkin13/spring-petclinic")
                }
            }
        }
			
        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
   
        stage ('Deploy') {
            steps {
                input 'Deploy to Production?'
                milestone(1)
                script {
                         sh 'ansible-playbook -i inv.ini pre_deploy_prod.yml'
                         sh 'ansible-playbook -i inv.ini deploy_prod.yml'
				}
            }
        }
    }
}

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
   
        stage('DeployToProduction') {
            steps {
                input 'Deploy to Production?'
                milestone(1)
				sshagent(credentials : ['prod_login']) {
					script {
                        sh "sshpass -v ssh -o StrictHostKeyChecking=no ec2-user@$prod_ip \"docker pull pushkin13/spring-petclinic:${env.BUILD_NUMBER}\""
                        try {
                            sh "sshpass -v ssh -o StrictHostKeyChecking=no ec2-user@$prod_ip \"docker stop train-schedule\""
                            sh "sshpass -v ssh -o StrictHostKeyChecking=no ec2-user@$prod_ip \"docker rm train-schedule\""
                        } catch (err) {
                            echo: 'caught error: $err'
                        }
                        sh "sshpass -v ssh -o StrictHostKeyChecking=no ec2-user@$prod_ip \"docker run --restart always --name train-schedule -p 8080:8080 -d pushkin13/spring-petclinic:${env.BUILD_NUMBER}\""
                    }
                }
            }
	}
    }
}

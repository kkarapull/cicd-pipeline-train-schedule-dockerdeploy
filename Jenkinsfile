pipeline {
    agent any
    
    tools { 
	// Global tools to be used by the pipeline
     //   maven 'maven3.8.4' 
        jdk 'jdk9' 
    }
        
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker Image') {
           // when {
            //    branch 'example-solution'
         //   }
            steps {
                script {
                    app = docker.build("kkarapull/train-schedule")
                    app.inside {
                        sh 'echo $(curl localhost:8888)'
                    }
                }
            }
        }
        stage('Push Docker Image') {
          when {
                branch 'example-solution'
           }
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
            when {
                branch 'example-solution'
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                    script {
                        sh "USERPASS="^yK*)0[w" && sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@44.203.9.124 \"docker pull kkarapull/train-schedule:${env.BUILD_NUMBER}\""
                        try {
                            sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@44.203.9.124 \"docker stop train-schedule\""
                            sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@44.203.9.124 \"docker rm train-schedule\""
                        } catch (err) {
                            echo: 'caught error: $err'
                        }
                        sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@44.203.9.124 \"docker run --restart always --name train-schedule -p 8080:8080 -d kkarapull/train-schedule:${env.BUILD_NUMBER}\""
                    }
                }
            }
        }
    }
}

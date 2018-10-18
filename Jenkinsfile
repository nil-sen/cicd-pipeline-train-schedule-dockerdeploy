pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    app = docker.build("nilsen12/train-schedule")
                    app.inside {
                        //sh 'echo $(curl localhost:8080)'
                    }
                }
            }
        }
        stage('Push Docker Image') {
            when {
                branch 'master'
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
        stage('Build Docker Image Private Registry') {
            when {
                branch 'master'
            }
            steps {
                script {
                    app = docker.build("train-schedule")
                    app.inside {
                        sh 'echo $(curl localhost:8080)'
                    }
                }
            }
        }
        stage('Push Docker Image Private Registry') {
            when {
                branch 'master'
            }
            steps {
                script {
                    docker.withRegistry("${dockerprivateregistry}",'') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        /*stage('DeployToProduction') {
            when {
                branch 'master'
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                withCredentials(bindings: [sshUserPrivateKey(credentialsId: 'CentOS_EC2_login', keyFileVariable: 'EC2_login', passphraseVariable: '', usernameVariable: 'centos')]){
                    script {
                        sh "ssh -i $keyfile -o StrictHostKeyChecking=no $username@$prod_ip \"sudo docker pull nilsen12/train-schedule:${env.BUILD_NUMBER}\""
                        try {
                            sh "ssh -i $keyfile -o StrictHostKeyChecking=no $username@$prod_ip \"sudo docker stop train-schedule\""
                            sh "ssh -i $keyfile -o StrictHostKeyChecking=no $username@$prod_ip \"sudo docker rm train-schedule\""
                        } catch (err) {
                            echo: 'caught error: $err'
                        }
                        sh "ssh -i $keyfile -o StrictHostKeyChecking=no $username@$prod_ip \"sudo docker run --restart always --name train-schedule -p 8080:8080 -d nilsen12/train-schedule:${env.BUILD_NUMBER}\""
                    }
                }
            }
        }*/
        stage('DeployToProduction') {
            when {
                branch 'master'
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                withCredentials(bindings: [sshUserPrivateKey(credentialsId: 'CentOS_EC2_login', keyFileVariable: 'EC2_login', passphraseVariable: '', usernameVariable: 'centos')]){
                    script {
                        sh "ssh -i $keyfile -o StrictHostKeyChecking=no $username@$prod_ip \"sudo docker pull $dockerprivaterepository/train-schedule:${env.BUILD_NUMBER}\""
                        try {
                            sh "ssh -i $keyfile -o StrictHostKeyChecking=no $username@$prod_ip \"sudo docker stop train-schedule\""
                            sh "ssh -i $keyfile -o StrictHostKeyChecking=no $username@$prod_ip \"sudo docker rm train-schedule\""
                        } catch (err) {
                            echo: 'caught error: $err'
                        }
                        sh "ssh -i $keyfile -o StrictHostKeyChecking=no $username@$prod_ip \"sudo docker run --restart always --name train-schedule -p 8080:8080 -d $dockerprivaterepository/train-schedule:${env.BUILD_NUMBER}\""
                    }
                }
            }
        }
    }
}

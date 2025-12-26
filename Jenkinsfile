pipeline {
    agent any

    environment {
        DOCKER_HOST = "tcp://host.docker.internal:2375"
        DOCKER_IMAGE_NAME = "mounikagorla/train-schedule"
        BRANCH_NAME = "main"
    }

    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }

        stage('Docker Test') {
            steps {
                sh 'docker ps'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    app = docker.build("${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER}")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-creds') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }

        stage('Debug Branch') {
            steps {
                echo "Branch detected: ${env.BRANCH_NAME}"
            }
        }

        stage('CanaryDeploy') {
          steps {
                echo "Branch detected"
            }
        }

        stage('DeployToProduction') {
            steps {
                echo "Branch detected"
            }
        }
    }
}

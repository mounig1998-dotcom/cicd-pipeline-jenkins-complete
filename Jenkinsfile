pipeline {
    agent any

    environment {
        DOCKER_IMAGE_NAME = "mounikagorla/train-schedule"
        DOCKER_HOST = "tcp://localhost:2375"
    }

    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage ('check') {
               steps{
                   sh 'which docker || echo "docker not found"'
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
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub-creds') {
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
            when {
                expression { env.BRANCH_NAME == 'master' || env.BRANCH_NAME == 'main' }
            }
            environment {
                CANARY_REPLICAS = 1
            }
            steps {
                script {
                    docker.image('bitnami/kubectl:latest').inside {
                        sh """
                            sed -i "s|REPLACE_IMAGE|${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER}|g" train-schedule-kube-canary.yml > prod-canary-updated.yml
                            kubectl apply -f prod-canary-updated.yml
                        """
                    }
                }
            }
        }

        stage('DeployToProduction') {
            when {
                expression { env.BRANCH_NAME == 'master' || env.BRANCH_NAME == 'main' }
            }
            environment {
                CANARY_REPLICAS = 0
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)

                script {
                    sh """
                        sed 's|\\\${DOCKER_IMAGE_NAME}|${DOCKER_IMAGE_NAME}|g; s|\\\${BUILD_NUMBER}|${env.BUILD_NUMBER}|g' train-schedule-kube.yml > prod-updated.yml
                        sed 's|\\\${DOCKER_IMAGE_NAME}|${DOCKER_IMAGE_NAME}|g; s|\\\${BUILD_NUMBER}|${env.BUILD_NUMBER}|g' train-schedule-kube-canary.yml > prod-canary-updated.yml
                    """

                    docker.image('bitnami/kubectl:latest').inside('--entrypoint=""') {
                        sh '''
                            echo "$KUBECONFIG_CONTENT" > ~/.kube/config
                            kubectl apply -f prod-canary-updated.yml
                            kubectl apply -f prod-updated.yml
                        '''
                    }
                }
            }
        }
    }
}

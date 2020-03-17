pipeline {
    agent any
    options {
        skipStagesAfterUnstable()
    }
    stages {
        stage('Pre Flight') {
            steps {
                script {
                    echo "The build number is ${env.BUILD_NUMBER}"
                    echo "The branch name is ${env.BRANCH_NAME}"
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    nginxImage = docker.build("thuyqnguyen/my-nginx:${env.BRANCH_NAME}-${env.BUILD_NUMBER}")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'my_docker_hub') {
                        nginxImage.push("${env.BRANCH_NAME}-${env.BUILD_NUMBER}")
                    }    
                }
            }
        }

        stage('Deploy to test') {
            when {
                branch 'test'
            }
            steps {
                script {
                    ssh '''
                        echo "deploy to ${test}"
                        ssh -o StrictHostKeyChecking=no cloud_user@${test} "docker image pull thuyqnguyen/my-nginx:${env.BRANCH_NAME}-${env.BUILD_NUMBER}"
                        try {
                            ssh -o StrictHostKeyChecking=no cloud_user@${test} "docker container stop my-nginx-${env.BRANCH_NAME}"
                            ssh -o StrictHostKeyChecking=no cloud_user@${test} "docker container rm my-nginx-${env.BRANCH_NAME}"
                        } catch (err) {
                            echo: 'caught error: $err'
                        ssh -o StrictHostKeyChecking=no cloud_user@${test} "docker run -d -p 8000:80 --name my-nginx-${env.BRANCH_NAME} thuyqnguyen/my-nginx:${env.BRANCH_NAME}-${env.BUILD_NUMBER}"
                    '''  
                }
            }
        }

        stage('Deploy to production') {
            when {
                branch 'master'
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                script {
                    ssh '''
                        ssh -o StrictHostKeyChecking=no cloud_user@${prod} "docker image pull thuyqnguyen/my-nginx:${env.BRANCH_NAME}-${env.BUILD_NUMBER}"
                        try {
                            ssh -o StrictHostKeyChecking=no cloud_user@${prod} "docker container stop my-nginx-${env.BRANCH_NAME}"
                            ssh -o StrictHostKeyChecking=no cloud_user@${prod} "docker container rm my-nginx-${env.BRANCH_NAME}"
                        } catch (err) {
                            echo: 'caught error: $err'
                        ssh -o StrictHostKeyChecking=no cloud_user@${prod} "docker run -d -p 8000:80 --name my-nginx-${env.BRANCH_NAME} thuyqnguyen/my-nginx:${env.BRANCH_NAME}-${env.BUILD_NUMBER}"
                    '''  
                }
            }
        }

    }
}

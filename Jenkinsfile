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
                    sh '''
                        echo "deploy to ${docker_test_ip}"
                        withCredentials([usernamePassword(credentialsId: 'docker_deploy', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                            echo ${USERNAME}
                            echo ${USERPASS}
                        }
                        /*---
                        withCredentials([usernamePassword(credentialsId: 'docker_deploy', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                            ssh -o StrictHostKeyChecking=no cloud_user@${docker_test_ip} "docker image pull thuyqnguyen/my-nginx:${env.BRANCH_NAME}-${env.BUILD_NUMBER}"
                            try {
                                ssh -o StrictHostKeyChecking=no -p ${USERPASS} ${USERPASS}@${docker_test_ip} "docker container stop my-nginx-${env.BRANCH_NAME}"
                                ssh -o StrictHostKeyChecking=no -p ${USERPASS} ${USERPASS}@${docker_test_ip} "docker container rm my-nginx-${env.BRANCH_NAME}"
                            } catch (err) {
                                echo: 'caught error: $err'
                            ssh -o StrictHostKeyChecking=no -p ${USERPASS} ${USERPASS}@${docker_test_ip} "docker run -d -p 8000:80 --name my-nginx-${env.BRANCH_NAME} thuyqnguyen/my-nginx:${env.BRANCH_NAME}-${env.BUILD_NUMBER}"
                        }
                        ---*/
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
                    sh '''
                        echo "deploy to ${docker_prod_ip}"
                        withCredentials([usernamePassword(credentialsId: 'docker_deploy', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                            ssh -o StrictHostKeyChecking=no cloud_user@${docker_prod_ip} "docker image pull thuyqnguyen/my-nginx:${env.BRANCH_NAME}-${env.BUILD_NUMBER}"
                            try {
                                ssh -o StrictHostKeyChecking=no -p ${USERPASS} ${USERPASS}@${docker_prod_ip} "docker container stop my-nginx-${env.BRANCH_NAME}"
                                ssh -o StrictHostKeyChecking=no -p ${USERPASS} ${USERPASS}@${docker_prod_ip} "docker container rm my-nginx-${env.BRANCH_NAME}"
                            } catch (err) {
                                echo: 'caught error: $err'
                            ssh -o StrictHostKeyChecking=no -p ${USERPASS} ${USERPASS}@${docker_prod_ip} "docker run -d -p 8000:80 --name my-nginx-${env.BRANCH_NAME} thuyqnguyen/my-nginx:${env.BRANCH_NAME}-${env.BUILD_NUMBER}"
                        }
                    '''  
                }
            }
        }

    }
}

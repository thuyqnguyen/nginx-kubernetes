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

        def test_host = [:]
        test_host.name = "test"
        test_host.host = "${docker_test_ip}"
        test_host.allowAnyHosts = true

        stage('Deploy to test') {
            when {
                branch 'test'
            }

            steps {
                withCredentials([usernamePassword(credentialsId: 'docker_deploy', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                    test_host.user = USERNAME
                    test_host.password = USERPASS
                    script {
                        sshCommand remote: test_host, command: "docker image pull thuyqnguyen/my-nginx:${env.BRANCH_NAME}-${env.BUILD_NUMBER}"
                        sshCommand remote: test_host, command: "pwd;ls"
                        /*
                        try {
                            ssh -o StrictHostKeyChecking=no -p ${USERPASS} ${USERNAME}@${docker_test_ip} "docker container stop my-nginx-${env.BRANCH_NAME}"
                            ssh -o StrictHostKeyChecking=no -p ${USERPASS} ${USERNAME}@${docker_test_ip} "docker container rm my-nginx-${env.BRANCH_NAME}"
                        } catch (err) {
                            echo: 'caught and ignore error: $err'
                        ssh -o StrictHostKeyChecking=no -p ${USERPASS} ${USERNAME}@${docker_test_ip} "docker run -d -p 8000:80 --name my-nginx-${env.BRANCH_NAME} thuyqnguyen/my-nginx:${env.BRANCH_NAME}-${env.BUILD_NUMBER}"
                        */
                    }
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

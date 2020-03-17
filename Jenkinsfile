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
                    stage ('ssh to test') {
                        def test_host = [:]
                        test_host.name = "test"
                        test_host.host = "${docker_test_ip}"
                        test_host.allowAnyHosts = true

                        withCredentials([usernamePassword(credentialsId: 'docker_deploy', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                            test_host.user = USERNAME
                            test_host.password = USERPASS
                                
                            sshCommand remote: test_host, command: "docker image pull thuyqnguyen/my-nginx:${env.BRANCH_NAME}-${env.BUILD_NUMBER}"
                            try {
                                sshCommand remote: test_host, command: "docker container stop my-nginx-${env.BRANCH_NAME}"
                                sshCommand remote: test_host, command: "docker container rm my-nginx-${env.BRANCH_NAME}"
                            }
                            catch(Exception e) {
                                echo "catch and ignore this error: ${e}"
                            }
                            sshCommand remote: test_host, command: "docker container run -d -p 8000:80 --name my-nginx-${env.BRANCH_NAME} thuyqnguyen/my-nginx:${env.BRANCH_NAME}-${env.BUILD_NUMBER}"                       
                        }
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
                    stage ('ssh to prod') {
                        def prod_host = [:]
                        prod_host.name = "prod"
                        prod_host.host = "${docker_prod_ip}"
                        prod_host.allowAnyHosts = true

                        withCredentials([usernamePassword(credentialsId: 'docker_deploy', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                            prod_host.user = USERNAME
                            prod_host.password = USERPASS
                                
                            sshCommand remote: prod_host, command: "docker image pull thuyqnguyen/my-nginx:${env.BRANCH_NAME}-${env.BUILD_NUMBER}"
                            try {
                                sshCommand remote: prod_host, command: "docker container stop my-nginx-${env.BRANCH_NAME}"
                                sshCommand remote: prod_host, command: "docker container rm my-nginx-${env.BRANCH_NAME}"
                            }
                            catch(Exception e) {
                                echo "catch and ignore this error: ${e}"
                            }
                            sshCommand remote: prod_host, command: "docker container run -d -p 8000:80 --name my-nginx-${env.BRANCH_NAME} thuyqnguyen/my-nginx:${env.BRANCH_NAME}-${env.BUILD_NUMBER}"                       
                        }
                    }
                }
            }
        }

    }
}

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
            /*---
            environment {
                my_docker_hub_url = 'https://docker.io'
            }
            steps {
                script {
                    withCredentials([usernamePassword( credentialsId: 'my_docker_hub', usernameVariable: 'my_docker_hub_user', passwordVariable: 'my_docker_hub_pass')]) {
                        nginxImage = docker.build("thuyqnguyen/my-nginx:${env.BRANCH_NAME}-${env.BUILD_NUMBER}")
                        sh "docker login -u ${my_docker_hub_user} -p ${my_docker_hub_pass} ${my_docker_hub_url}"
                        nginxImage.push()
                    }
                }
            }
            ---*/
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
                    app.push("${env.BRANCH_NAME}-${env.BUILD_NUMBER}")
                        
                }
            }
        }
        /*---
        stage('Test') {
            agent {
                docker {
                    image 'qnib/pytest'
                }
            }
            steps {
                sh 'py.test --verbose --junit-xml test-reports/results.xml sources/test_calc.py'
            }
            post {
                always {
                    junit 'test-reports/results.xml'
                }
            }
        }
        stage('Deliver') { 
            agent {
                docker {
                    image 'cdrx/pyinstaller-linux:python2' 
                }
            }
            steps {
                sh 'pyinstaller --onefile sources/add2vals.py' 
            }
            post {
                success {
                    archiveArtifacts 'dist/add2vals' 
                }
            }
        }
        ---*/
    }
}


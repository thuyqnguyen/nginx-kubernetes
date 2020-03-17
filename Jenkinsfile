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
            when {
                branch 'master'
            }
            ---*/
            steps {
                script {
                    docker.withRegistry('https://hub.docker.com') {
                        nginxImage = docker.build("thuyqnguyen/my-nginx:${env.BRANCH_NAME}-${env.BUILD_NUMBER}")
                        nginxImage.inside {
                            sh 'echo $(curl localhost:80)'
                        }
                        nginxImage.push()
                    }
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


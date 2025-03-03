pipeline {
    agent any

    environment {
        DOCKER_PROD_REPO = 'devopss1234/prod'
        DOCKER_CREDENTIALS_ID = 'dockerhub-credentials'
        GIT_CREDENTIALS_ID = 'git-credentials'
        GIT_REPO_URL = 'https://github.com/Pradeepans/FinalProject.git'
        BRANCH_NAME = 'master'
    }

    stages {
        stage('Checkout Code') {
            steps {
                script {
                    checkout([
                        $class: 'GitSCM',
                        branches: [[name: "*/${BRANCH_NAME}"]],
                        userRemoteConfigs: [[
                            url: "${GIT_REPO_URL}",
                            credentialsId: "${GIT_CREDENTIALS_ID}"
                        ]]
                    ])
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                script {
                    sh './build.sh'
                }
            }
        }

        stage('Push Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    def dockerRepo = DOCKER_PROD_REPO
                    def buildTag = "${dockerRepo}:${env.BUILD_NUMBER}"
                    sh "docker build -t ${buildTag} ."
                    withCredentials([usernamePassword(credentialsId: DOCKER_CREDENTIALS_ID, usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh "echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin"
                    }
                    sh "docker push ${buildTag}"
                }
            }
        }

        stage('Deploy Docker Containers') {
            steps {
                script {
                    def buildTag = "${DOCKER_PROD_REPO}:${env.BUILD_NUMBER}"
                    sh "docker pull ${buildTag}"
                    sh './deploy.sh'
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline executed successfully!'
        }
        failure {
            echo 'Pipeline execution failed.'
        }
    }
}


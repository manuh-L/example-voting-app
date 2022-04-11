pipeline {
    agent none

    stages {
        stage('worker build') {
            agent {
                docker {
                    image 'maven:3.8-openjdk-11-slim'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }

            when {
                changeset '**/worker/**' //when code chanfe on worker
            }
            steps {
                echo 'Compiling worker app'
                dir('worker') {
                    sh 'mvn compile'
                }
            }
        }
        stage('worker test') {
            agent {
                docker {
                    image 'maven:3.8-openjdk-11-slim'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }

            when {
                changeset '**/worker/**' //when code chanfe on worker
            }
            steps {
                echo 'Running Unit Tests on worker app'
                dir('worker') {
                    sh 'mvn clean test'
                }
            }
        }
        stage('worker package') {
            agent {
                docker {
                    image 'maven:3.8-openjdk-11-slim'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }

            when {
                branch 'master'
                changeset '**/worker/**' //when code change on worker
            }
            steps {
                echo 'Packaging worker app'
                dir('worker') {
                    sh 'mvn package -DskipTests'
                    archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
                }
            }
        }

        stage('worker-docker-package') {
            agent any
            when {
                branch 'master'
                changeset '**/worker/**' //when code chanfe on worker
            }
            steps {
                echo 'Packaging worker app with docker'
                script {
                        docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
                        def workerImage = docker.build("nhiuana/worker:v${env.BUILD_ID}", './worker') //build the image with build number as tag

                        /* Push the container to the custom Registry */
                        workerImage.push()
                        workerImage.push("${env.BRANCH_NAME}") //push the image to docker hub with branch name as tag env.BRANCH_NAME
                        }
                }
            }
        }

        stage('result-build') {
            agent {
                docker {
                    image 'node:8.16.0-alpine'
                }
            }
            when {
                changeset '**/result/**' //when code chanfe on worker
            }
            steps {
                echo 'Compiling result application'
                dir('result') {
                    sh 'npm install'
                    sh 'npm ls'
                }
            }
        }
        stage('result-test') {
            agent {
                docker {
                    image 'node:8.16.0-alpine'
                }
            }
            when {
                changeset '**/result/**' //when code change on worker
            }
            steps {
                echo 'Running Unit Tests on result application'
                dir('result') {
                    sh 'npm install'
                    sh 'npm test'
                }
            }
        }

        stage('result-docker-package') {
            agent any
            when {
                branch 'master'
                changeset '**/result/**' //when code chanfe on worker
            }
            steps {
                echo 'Packaging result app with docker'
                script {
                        docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
                        def workerImage = docker.build("nhiuana/result:v${env.BUILD_ID}", './result') //build the image with build number as tag

                        /* Push the container to the custom Registry */
                        workerImage.push()
                        workerImage.push("${env.BRANCH_NAME}") //push the image to docker hub with branch name as tag
                        }
                }
            }
        }

        stage('vote-build') {
            agent {
                docker {
                    image 'python:2.7.16-slim'
                    args '--user root'
                }
            }
            when {
                changeset '**/vote/**' //when code chanfe on worker
            }
            steps {
                echo 'Compiling vote app'
                dir('vote') {
                    sh 'pip install -r requirements.txt'
                }
            }
        }
        stage('vote-test') {
            agent {
                docker {
                    image 'python:2.7.16-slim'
                    args '--user root'
                }
            }
            when {
                changeset '**/vote/**' //when code change on worker
            }
            steps {
                echo 'Running Unit Tests on vote application'
                dir('vote') {
                    sh 'pip install -r requirements.txt'
                    sh 'nosetests -v'
                }
            }
        }

        stage('vote-docker-package') {
            agent any
            when {
                branch 'master'
                changeset '**/vote/**' //when code chanfe on worker
            }
            steps {
                echo 'Packaging vote app with docker'
                script {
                        docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
                        def workerImage = docker.build("nhiuana/vote:v${env.BUILD_ID}", './vote') //build the image with build number as tag

                        /* Push the container to the custom Registry */
                        workerImage.push()
                        workerImage.push("${env.BRANCH_NAME}") //push the image to docker hub with branch name as tag env.BRANCH_NAME
                        }
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline for instavote app worker app is complete'
        }
        failure {
            slackSend (channel: 'instavote-cd', message: "Build Failed - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
        }
        success {
            slackSend (channel: 'instavote-cd', message: "Build Succeed - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
        }
    }
}

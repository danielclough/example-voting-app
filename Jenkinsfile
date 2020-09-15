pipeline {

    agent none 

  // Worker Pipeline
    
    stages{
        stage('worker-build'){

            agent{
                docker{
                    image 'maven:3.6.1-jdk-8-slim'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }

            when { changeset "**/worker/**" }
            steps{
                echo 'Compiling'
                dir('worker'){
                    sh 'mvn -e compile'}
            }
        }
        stage('worker-test'){

            agent{
                docker{
                    image 'maven:3.6.1-jdk-8-slim'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }

            when { changeset "**/worker/**" }
            steps{
                echo 'Testing'              
                dir('worker'){
                    sh 'mvn -e clean test'}
            }
        }
        stage('worker-docker-package'){

            agent any

            when { 
                changeset "**/worker/**" 
                branch 'master'
            }
            steps{
                echo 'Packaging with Docker'       
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'pwDocker') {
                        def workerImage = docker.build("danielclough/worker:v${env.BUILD_ID}", "./worker")
                        workerImage.push()
                        workerImage.push("${env.BRANCH_NAME}")
                    }
                }
            }
        }

  // Result Pipeline

        stage('result-build'){
            agent{
                docker{
                    image 'node:8.9-alpine'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
            when { changeset "**/result/**" }
            steps{
                echo 'Building'
                dir('result'){
                    sh 'npm install'
                }
            }
        }
        stage('result-test'){
            agent{
                docker{
                    image 'node:8.9-alpine'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
            when { changeset "**/result/**" }
            steps{
                echo 'Testing'
                dir('result'){    
                    sh 'npm install'
                    sh 'npm test'
                }
            }
        }
        stage('result-docker-package'){
            agent any
            when { 
                changeset "**/result/**" 
                branch 'master'
            }
            steps{
                echo 'Packaging with Docker'       
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'pwDocker') {
                        def resultImage = docker.build("danielclough/result:v${env.BUILD_ID}", "./result")
                        resultImage.push()
                        resultImage.push("${env.BRANCH_NAME}")
                    }
                }
            }
        }


  // Vote Pipeline

        stage('vote-build'){
            agent{
                docker{
                    image 'python:2.7.16-slim'
                    args '-v $HOME/.m2:/root/.m2'
                    args '--user root'
                }
            }
            when { changeset "**/vote/**" }
            steps{
                echo 'Compiling'
                dir('vote'){
                    sh 'pip install -r requirements.txt'
                }
            }
        }

        stage('vote-test'){
            agent{
                docker{
                    image 'python:2.7.16-slim'
                    args '-v $HOME/.m2:/root/.m2'
                    args '--user root'
                }
            }
            when { changeset "**/vote/**" }
            steps{
                echo 'Testing'              
                dir('vote'){
                    sh 'pip install -r requirements.txt'
                    sh 'nosetests -v'
                }
            }
        }

        stage('vote-docker-package'){
            agent any
            when { 
                changeset "**/vote/**" 
                branch 'master'
            }
            steps{
                echo 'Packaging with Docker'       
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'pwDocker') {
                        def voteImage = docker.build("danielclough/vote:v${env.BUILD_ID}", "./vote")
                        voteImage.push()
                        voteImage.push("${env.BRANCH_NAME}")
                    }
                }
            }
        }
        stage('deploy-to-dev'){
            agent any
            when { 
                branch 'master'
            }
            steps{
                echo 'Deply to Dev'       
                sh 'docker-compose up -d'
            }

        }

    }
    post{
        always{
            echo 'Complete!'
        }
    }
}
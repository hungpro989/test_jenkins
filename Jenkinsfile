pipeline {

    agent any

    tools {
        maven 'my-maven'
    }
    stages {

        stage('Build with Maven') {
            steps {
                sh 'mvn --version'
                sh 'java -version'
                sh 'mvn clean package -Dmaven.test.failure.ignore=true'
            }
        }

        stage('Packaging/Pushing imagae') {

            steps {
                withDockerRegistry(credentialsId: 'dockerhub', url: 'https://index.docker.io/v1/') {
                    sh 'docker build -t hungpro989/jenkins_demo .'
                    sh 'docker push hungpro989/jenkins_demo'
                }
            }
        }

        stage('Deploy Spring Boot to DEV') {
            steps {
                echo 'Deploying and cleaning'
                sh 'docker image pull hungpro989/jenkins_demo'
                sh 'docker container stop jenkins-demo-springboot || echo "this container does not exist" '
                sh 'docker network create dev || echo "this network exists"'
                sh 'echo y | docker container prune '

                sh 'docker container run -d --rm --name jenkins-demo-springboot -p 8887:8887 --network dev hungpro989/jenkins_demo'
            }
        }

    }
    post {
        // Clean after build
        always {
            cleanWs()
        }
    }
}
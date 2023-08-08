pipeline {

    agent any

    tools {
        maven 'my-maven'
    }
    environment {
        MYSQL_ROOT_LOGIN = credentials('mysql-root')
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
                    sh 'docker build -t hungpro989/koroapp .'
                    sh 'docker push hungpro989/koroapp'
                }
            }
        }

        stage('Deploy MySQL to DEV') {
            steps {
                echo 'Deploying and cleaning'
                sh 'docker image pull mysql:8.0'
                sh 'docker network create dev || echo "this network exists"'
                sh 'docker container stop koro-mysql || echo "this container does not exist" '
                sh 'echo y | docker container prune '
                sh 'docker volume rm koro-mysql-data || echo "no volume"'

                sh "docker run --name koro-mysql --rm --network dev -v koro-mysql-data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=n01s4ud3mv4ng -e MYSQL_DATABASE=koro_shop  -d mysql:8.0"
                sh 'sleep 20'
                sh "docker exec -i koro-mysql mysql --user=root --password=n01s4ud3mv4ng < script"
            }
        }

        stage('Deploy Spring Boot to DEV') {
            steps {
                echo 'Deploying and cleaning'
                sh 'docker image pull hungpro989/koroapp'
                sh 'docker container stop koroapp-springboot || echo "this container does not exist" '
                sh 'docker network create dev || echo "this network exists"'
                sh 'echo y | docker container prune '

                sh 'docker container run -d --rm --name koroapp-springboot -p 8081:8080 --network dev hungpro989/koroapp'
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
pipeline {

    agent any

    environment {
        MYSQL_ROOT_LOGIN = credentials('mysql')
    }
    stages {

        stage('Packaging/Pushing imagae') {

            steps {
                withDockerRegistry(credentialsId: 'dockerhub', url: 'https://index.docker.io/v1/') {
                    sh 'docker build -t duytan/ci-cd .'
                    sh 'docker push duytan/ci-cd'
                }
            }
        }

        stage('Deploy MySQL to DEV') {
            steps {
                echo 'Deploying and cleaning'
                sh 'docker image pull mysql:8.0'
                sh 'docker network create dev || echo "this network exists"'
                sh 'docker container stop duytan-mysql || echo "this container does not exist" '
                sh 'echo y | docker container prune '
                sh 'docker volume rm duytan-mysql-data || echo "no volume"'

                sh "docker run --name duytan-mysql --rm --network dev -v duytan-mysql-data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_LOGIN_PSW} -e MYSQL_DATABASE=db_example  -d mysql:8.0 "
                sh 'sleep 20'
                sh "docker exec -i duytan-mysql mysql --user=root --password=${MYSQL_ROOT_LOGIN_PSW} < script"
            }
        }

        stage('Deploy Spring Boot to DEV') {
            steps {
                echo 'Deploying and cleaning'
                sh 'docker image pull duytan/ci-cd'
                sh 'docker container stop duytan-ci-cd || echo "this container does not exist" '
                sh 'docker network create dev || echo "this network exists"'
                sh 'echo y | docker container prune '

                sh 'docker container run -d --rm --name duytan-ci-cd -p 8081:8080 --network dev duytan/ci-cd'
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

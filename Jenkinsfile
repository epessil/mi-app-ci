pipeline {
    agent any

    environment {
        IMAGE = 'flacuss74/mi-app'
        TAG   = "2.${BUILD_NUMBER}"
    }

    stages {
        stage('Build') {
            steps {
                sh 'docker build -t $IMAGE:$TAG .'
            }
        }

        stage('Test') {
            steps {
                sh 'docker run -d --name test-app $IMAGE:$TAG'
                sh 'sleep 5'
                sh '''docker exec test-app python3 -c "import urllib.request,sys; r=urllib.request.urlopen('http://localhost:8080/health').read(); sys.exit(0 if b'ok' in r else 1)"'''
            }
        }

        stage('Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-flacuss74', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_TOKEN')]) {
                    sh 'echo $DOCKER_TOKEN | docker login -u $DOCKER_USER --password-stdin'
                    sh 'docker push $IMAGE:$TAG'
                }
            }
        }
    }

    post {
        always {
            sh 'docker rm -f test-app || true'
            sh 'docker logout || true'
        }
    }
}

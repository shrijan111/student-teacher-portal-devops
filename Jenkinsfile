pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                git 'https://github.com/shrijan111/student-teacher-portal-devops.git'
            }
        }

        stage('Build Backend Image') {
            steps {
                sh 'docker build -t la000la/student-portal-backend:${BUILD_NUMBER} ./backend'
            }
        }

        stage('Build Frontend Image') {
            steps {
                sh '''
                    docker build \
                      -t la000la/student-portal-frontend:${BUILD_NUMBER} \
                      --build-arg REACT_APP_API_BASE_URL=http://192.168.56.103:3500 \
                      ./frontend
                '''
            }
        }

        stage('Push Images to Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub',
                        usernameVariable: 'DOCKERHUB_USERNAME',
                        passwordVariable: 'DOCKERHUB_TOKEN'
                    )
                ]) {
                    sh '''
                        echo "$DOCKERHUB_TOKEN" | docker login \
                          -u "$DOCKERHUB_USERNAME" \
                          --password-stdin

                        docker push la000la/student-portal-backend:${BUILD_NUMBER}
                        docker push la000la/student-portal-frontend:${BUILD_NUMBER}

                        docker logout
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    kubectl -n student-portal set image deployment/backend \
                      backend=la000la/student-portal-backend:${BUILD_NUMBER}

                    kubectl -n student-portal set image deployment/frontend \
                      frontend=la000la/student-portal-frontend:${BUILD_NUMBER}

                    kubectl -n student-portal rollout status deployment/backend
                    kubectl -n student-portal rollout status deployment/frontend
                '''
            }
        }
    }
}

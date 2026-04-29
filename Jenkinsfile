pipeline {
    agent any

    triggers {
        githubPush()
    }

    options {
        disableConcurrentBuilds()
    }

    environment {
        APP_NAME = "todo-app"
        PORT = "3005"
        IMAGE_TAG = "v${BUILD_NUMBER}"
    }

    stages {

        stage('Clone Repo') {
            steps {
                deleteDir()
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[
                        url: 'https://github.com/sanchitpandit/getting-started-app.git'
                    ]]
                ])
            }
        }

        stage('Create Dockerfile') {
            steps {
                sh '''
                echo "FROM node:18-alpine" > Dockerfile
                echo "WORKDIR /app" >> Dockerfile
                echo "COPY . ." >> Dockerfile
                echo "WORKDIR /app/app" >> Dockerfile
                echo "RUN npm install" >> Dockerfile
                echo "ENV HOST=0.0.0.0" >> Dockerfile
                echo "EXPOSE 3000" >> Dockerfile
                echo "CMD [\\"npm\\", \\"run\\", \\"dev\\"]" >> Dockerfile
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t $APP_NAME:$IMAGE_TAG .
                docker tag $APP_NAME:$IMAGE_TAG $APP_NAME:latest
                '''
            }
        }

        stage('Deploy Container') {
            steps {
                sh '''
                docker stop $APP_NAME || true
                docker rm $APP_NAME || true
                docker run -d --restart=always -p $PORT:3000 --name $APP_NAME $APP_NAME:$IMAGE_TAG
                '''
            }
        }

        stage('Verify') {
            steps {
                sh '''
                sleep 5
                curl -f http://localhost:$PORT || exit 1
                '''
            }
        }

        stage('Cleanup') {
            steps {
                sh 'docker image prune -f'
            }
        }
    }
}

pipeline {
    agent any
    tools{
        maven "mvn"
    }
    environment{
        DOCKER_CREDS= credentials('dockerhub')
    }

    stages {
        stage('Initialize') {
            steps {
                sh '''
                  echo "PATH=${PATH}"
                  echo "M2_HOME= ${M2_HOME}"
                '''
            }
        }
        stage('Git Checkout') {
            steps {
                git(
                    url: 'https://github.com/leoinfnet/devops-cervejaria-acme.git',
                    branch: 'main')
                script{
                    TAG_VERSION = sh(script: 'git rev-parse --short HEAD', returnStdout:true).trim()
                }
            }
        }
        stage('Build') {
             steps{
                 sh "mvn -DskipTests clean package"

             }
             post{
                 success {
                     sh 'cp target/cervejaria-acme-0.0.1-SNAPSHOT.jar docker/'
                 }
             }
         }

          stage('Create Image') {
             steps{
                 echo "IMAGE VERSION: ${TAG_VERSION}"
                 sh "docker build . -f docker/Dockerfile -t leogloriainfnet/cervejas-complete:${TAG_VERSION}"
                 sh "docker tag leogloriainfnet/cervejas-complete:${TAG_VERSION} leogloriainfnet/cervejas-complete:latest"
             }

         }
         stage('Push Image') {
             steps{
                sh "docker login --username=$DOCKER_CREDS_USR --password=$DOCKER_CREDS_PSW"
                sh "docker push leogloriainfnet/cervejas-complete:${TAG_VERSION}"
                sh "docker push leogloriainfnet/cervejas-complete:latest "
             }

         }
         stage('Update K8S') {
             steps{
               sh "kubectl set image deploy/cervejaria-acme -n cervejaria cervejaria-acme=leogloriainfnet/cervejas-complete:$TAG_VERSION"
                sh 'kubectl rollout status deploy/cervejaria-acme -n cervejaria'
             }

         }
         stage('Clean Images') {
            steps{
                sh "docker image rm leogloriainfnet/cervejas-complete:${TAG_VERSION}"
                sh "docker image rm leogloriainfnet/cervejas-complete:latest"
            }

         }
    }
}

pipeline {
    agent any
    tools {
      jdk 'Java17'
      maven 'Maven3'
    }
    environment {
        APP_NAME = "java-application"
        RELEASE = "1.0.0"
        DOCKER_USER = "kushaggarwal"
        DOCKER_PASS = "dockerhub"
        IMAGE_NAME = "${DOCKER_USER}"+"/"+"${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
        JENKINS_API_TOKEN = credentials("JENKINS_API_TOKEN")
    }
    stages {
      stage("Clean Workspace"){
        steps{
          cleanWs()
        }
    }
    stage("Checkout from SCM"){
      steps {
        git branch: 'main', url: 'https://github.com/kushaggarwal/register-app.git'
    }
    }
    stage("Build Application"){
      steps{
        sh "mvn clean package"
      }
    }
      stage("Test Application"){
        steps {
          sh "mvn test"
        }
      }
    stage("SonarQube Analysis"){
        steps{
            script{
                withSonarQubeEnv(credentialId: 'Jenkins-sonarqube-token'){
                    sh "mvn sonar:sonar"
                }
            }
        }
    }
        stage("Build & Push Docker Image") {
            steps {
                script {
                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image = docker.build "${IMAGE_NAME}"
                    }

                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push('latest')
                    }
                }
            }

       }
                stage("Trigger CD pipeline job") {
            steps {
                script {
                    sh "curl -v -k --user kush:${JENKINS_API_TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' 'ec2-13-61-184-245.eu-north-1.compute.amazonaws.com:8080/job/gitops-cd/buildWithParameters?token=gitops-token'"
                }
            }

       }
          
}
}

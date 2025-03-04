pipeline {
  agent {label 'agent'}
  tools {
    jdk 'jdk17'
    maven 'maven3'
  }
  environment {
    APP_NAME = "register-app"
    RELEASE ="1.0"
    DOCKER_USER = "satyavenkat"
    DOCKER_PASS = "dockerhub"
    IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
    IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
    JENKINS_API_TOKEN = credentials("JENKINS_API_TOKEN")
  }
  stages {
    stage ("Clenup Workplace"){
      steps {
        cleanWs()
      } 
    }
    stage ("Checkout from SCM"){
      steps {
        git branch: 'main', 
        changelog: false, 
        credentialsId: 'github', 
        poll: false, 
        url: 'https://github.com/Satyavenkat2813/register-app-ss.git'
        
      }
    }
    stage ("Build Application"){
      steps {
        sh "mvn clean package"
      }  
    }
    stage ("Test Application"){
      steps {
        sh "mvn test"
      }
    }
    stage ("Sonarqube Analysis"){
      steps {
        script {
          withSonarQubeEnv(credentialsId: 'jenkins-sonar'){
            sh "mvn sonar:sonar "
          }
        }
      }
      
    }
    stage ("Quality Gate"){
      steps {
        script {
          waitForQualityGate abortPipeline: false, credentialsId: 'jenkins-sonar'
        }
        
      }
    }
    stage ("Build & Push Docker Image"){
      steps {
        script {
          docker.withRegistry('',DOCKER_PASS) {
            docker_image = docker.build "${IMAGE_NAME}"
          }
          docker.withRegistry ('',DOCKER_PASS){
            docker_image.push("${IMAGE_TAG}")
            docker_image.push('latest')
          }
        }
      }
    }
    stage ("Trivy scan"){
      steps {
        script {
          sh ('docker run -v  /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image satyavenkat/register-app:latest --no-progress --scanners vuln --exit-code 0 --severity HIGH,CRITICAL --format table')
        }
      }
    }
    stage ("Cleanup Artifacts"){
      steps {
        script{
          sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
          sh "docker rmi ${IMAGE_NAME}:latest"
        }
      }
    }
    stage(" Trigger CD Pipeline"){
      steps {
        script {
          sh "curl -v -k --user satya:${JENKINS_API_TOKEN} -X POST -H 'cache-contol: no-cache' -H 'content-type: application/x-www-from-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' 'ec2-13-127-22-126.ap-south-1.compute.amazonaws.com:8080/job/CD_Pipeline
/buildWithParameters?token=gitops-token'"
         }
      }
    }
  }
}

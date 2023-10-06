pipeline {
  agent {label 'agent'}
  tools {
    jdk 'jdk17'
    mavne 'mavne3'
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
  }
}

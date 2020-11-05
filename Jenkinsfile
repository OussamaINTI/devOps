pipeline {
 agent any
 environment {
  SONARQUBE_URL = "http://localhost"
  SONARQUBE_PORT = "9000"
 }
 options {
  skipDefaultCheckout()
 }
 stages {
  stage('SCM') {
   steps {
    checkout scm
   }
  }
  stage('Code Quality Analysis') {
   parallel {
    stage('SonarQube') {
     agent {
      docker {
       image 'maven:3.6.0-jdk-8-alpine'
       args "-v ${user.home}/.m2/repository/"
       reuseNode true
      }
     }
     steps {
      sh " mvn sonar:sonar -Dsonar.host.url=$SONARQUBE_URL:$SONARQUBE_PORT"
     }
    }
   }
  }
 }
}
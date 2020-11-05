pipeline {
 agent any
 environment {
  // This can be nexus3 or nexus2
  NEXUS_VERSION = "nexus3"
  // This can be http or https
  NEXUS_PROTOCOL = "http"
  // Where your Nexus is running. In my case:
  NEXUS_URL = "ec2-52-212-29-159.eu-west-1.compute.amazonaws.com:8081"
  // Repository where we will upload the artifact
  NEXUS_REPOSITORY = "maven-snapshots"
  // Jenkins credential id to authenticate to Nexus OSS
  NEXUS_CREDENTIAL_ID = "nexus-credentials"
  /* 
    Windows: set the ip address of docker host. In my case 192.168.99.100.
    to obtains this address : $ docker-machine ip
    Linux: set localhost to SONARQUBE_URL
  */
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
  stage("build & SonarQube analysis") {
    agent any
            steps {
              withSonarQubeEnv('My SonarQube Server') {
                sh 'mvn clean package sonar:sonar'
              }
            }
          }        
 }
}
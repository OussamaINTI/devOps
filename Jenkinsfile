node{
stage('SCM'){
 git 'https://github.com/OussamaINTI/devOps'
}
stage('Compile'){
def mvnHome = tool name: 'maven-3' , type: 'maven'
sh "${mvnHome}/bin/mvn package"
}
 stage("SonarQube Analysis") {
            def scannerHome = tool 'SonarQube Scanner 4';
            sh "${scannerHome}/bin/sonar-scanner"
      }
}
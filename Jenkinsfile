pipeline {
 agent any
 environment {
  SONARQUBE_URL = "http://localhost"
  SONARQUBE_PORT = "9000"
 }
 options {
  skipDefaultCheckout()
 }
 tools {
    maven 'maven'
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
     steps {
      sh " mvn sonar:sonar -Dsonar.host.url=$SONARQUBE_URL:$SONARQUBE_PORT"
     }
    }
	stage('SonarQube analysis') {
      steps {
        script {
          // requires SonarQube Scanner 2.8+
          scannerHome = tool 'SonarQube Scanner 4'
        }
        withSonarQubeEnv('SonarQube Scanner') {
          sh "${scannerHome}/bin/sonar-scanner"
        }
      }
    }
   }
  }
  stage('Deploy Artifact To Nexus') {
   when {
    anyOf { branch 'master'; branch 'develop' }
   }
   steps {
    script {
     unstash 'pom'
     unstash 'artifact'
     // Read POM xml file using 'readMavenPom' step , this step 'readMavenPom' is included in: https://plugins.jenkins.io/pipeline-utility-steps
     pom = readMavenPom file: "pom.xml";
     // Find built artifact under target folder
     filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
     // Print some info from the artifact found
     echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
     // Extract the path from the File found
     artifactPath = filesByGlob[0].path;
     // Assign to a boolean response verifying If the artifact name exists
     artifactExists = fileExists artifactPath;
     if (artifactExists) {
      nexusArtifactUploader(
       nexusVersion: NEXUS_VERSION,
       protocol: NEXUS_PROTOCOL,
       nexusUrl: NEXUS_URL,
       groupId: pom.groupId,
       version: pom.version,
       repository: NEXUS_REPOSITORY,
       credentialsId: NEXUS_CREDENTIAL_ID,
       artifacts: [
        // Artifact generated such as .jar, .ear and .war files.
        [artifactId: pom.artifactId,
         classifier: '',
         file: artifactPath,
         type: pom.packaging
        ],
        // Lets upload the pom.xml file for additional information for Transitive dependencies
        [artifactId: pom.artifactId,
         classifier: '',
         file: "pom.xml",
         type: "pom"
        ]
       ]
      )
     } else {
      error "*** File: ${artifactPath}, could not be found";
     }
    }
   }
  }
  stage('Deploy to Staging Servers') {
   when {
    anyOf { branch 'master'; branch 'develop' }
   }
   agent {
    docker {
     image 'ahmed24khaled/ansible-management'
     reuseNode true
    }
   }
   steps {
    script {

     pom = readMavenPom file: "pom.xml"
     repoPath = "${pom.groupId}".replace(".", "/") + "/${pom.artifactId}"
     version = pom.version
     artifactId = pom.artifactId
     withEnv(["ANSIBLE_HOST_KEY_CHECKING=False", "APP_NAME=${artifactId}", "repoPath=${repoPath}", "version=${version}"]) {
      sh '''
      
        curl --silent "http://$NEXUS_URL/repository/maven-snapshots/${repoPath}/${version}/maven-metadata.xml" > tmp &&
        egrep '<value>+([0-9\\-\\.]*)' tmp > tmp2 &&
        tail -n 1 tmp2 > tmp3 &&
        tr -d "</value>[:space:]" < tmp3 > tmp4 &&
        REPO_VERSION=$(cat tmp4) &&

        export APP_SRC_URL="http://${NEXUS_URL}/repository/maven-snapshots/${repoPath}/${version}/${APP_NAME}-${REPO_VERSION}.war" &&
        ansible-playbook -v -i ./ansible_provisioning/hosts --extra-vars "host=staging" ./ansible_provisioning/playbook.yml 

       '''
     }
    }
   }
  }
   stage('Deploy to Production Servers') {
   when {
    branch 'master'
   }
   agent {
    docker {
     image 'ahmed24khaled/ansible-management'
     reuseNode true
    }
   }
   steps {
    script {

     pom = readMavenPom file: "pom.xml"
     repoPath = "${pom.groupId}".replace(".", "/") + "/${pom.artifactId}"
     version = pom.version
     artifactId = pom.artifactId
     withEnv(["ANSIBLE_HOST_KEY_CHECKING=False", "APP_NAME=${artifactId}", "repoPath=${repoPath}", "version=${version}"]) {
      sh '''
      
        curl --silent "$NEXUS_URL/repository/maven-snapshots/${repoPath}/${version}/maven-metadata.xml" > tmp &&
        egrep '<value>+([0-9\\-\\.]*)' tmp > tmp2 &&
        tail -n 1 tmp2 > tmp3 &&
        tr -d "</value>[:space:]" < tmp3 > tmp4 &&
        REPO_VERSION=$(cat tmp4) &&

        export APP_SRC_URL="http://${NEXUS_URL}/repository/maven-snapshots/${repoPath}/${version}/${APP_NAME}-${REPO_VERSION}.war" &&
        ansible-playbook -v -i ./ansible_provisioning/hosts --extra-vars "host=production" ./ansible_provisioning/playbook.yml 

       '''
     }
    }
   }
  }
 }
}
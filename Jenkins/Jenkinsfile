def registry = 'https://paskou.jfrog.io/'
pipeline {
    agent {
        node {
            label 'maven'
        }
    }
environment 
{ 
    PATH = "/opt/apache-maven-3.8.8/bin: $PATH "
}
    stages {
        stage('Build-maven') {
            steps {
               sh 'mvn clean package'
            }
        }
        
stage('SonarQube analysis') {
    environment 
{ 
    scannerHome = tool 'nitro-sonar-scanner'
}
    steps {
              withSonarQubeEnv('nitro-sonarqube-server') { 
      sh "${scannerHome}/bin/sonar-scanner"
    } 
            }
    
  }
  
stage("Jar Publish") {
        steps {
            script {
                    echo '<--------------- Jar Publish Started --------------->'
                     def server = Artifactory.newServer url:registry+"/artifactory" ,  credentialsId:"jfrog-cred"
                     def properties = "buildid=${env.BUILD_ID},commitid=${GIT_COMMIT}";
                     def uploadSpec = """{
                          "files": [
                            {
                              "pattern": "target/(*)",
                              "target": "nitro-libs-release-local/{1}",
                              "flat": "false",
                              "props" : "${properties}",
                              "exclusions": [ "*.sha1", "*.md5"]
                            }
                         ]
                     }"""
                     def buildInfo = server.upload(uploadSpec)
                     buildInfo.env.collect()
                     server.publishBuildInfo(buildInfo)
                     echo '<--------------- Jar Publish Ended --------------->'  
            
            }
        }   
    }  
    }
}

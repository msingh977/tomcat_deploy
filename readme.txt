node {
   
    // agent {
    //     label 'jenkins-helper'
    // }
    
    stage("Email Notification"){
          slackSend baseUrl: 'https://hooks.slack.com/services/', 
          botUser: true, 
          channel: 'training', 
          color: 'green', 
          iconEmoji: ':)', 
          message: 'Jenkins is starting', 
          notifyCommitters: true, 
          teamDomain: '/opt/slack-emails', 
          tokenCredentialId: 'ID_SLACK', 
          username: 'msingh977'
    }
    
    stage("Git Clone"){
        git credentialsId: 'GIT_CREDENTIALS', url: 'https://github.com/msingh977/tomcat_deploy.git'
    }
    
      stage("Maven Clean Package"){
        def mvnHome =  tool name: 'Maven', type: 'maven'
        sh "${mvnHome}/bin/mvn clean package"
    }
     
      stage('SonarQube Analysis') {
        def mvnHome =  tool name: 'Maven', type: 'maven'
        withSonarQubeEnv('Sonarqube') { 
        sh "${mvnHome}/bin/mvn sonar:sonar"
        }
    }
    
     stage('Deploy to Nexus') {
        nexusArtifactUploader artifacts: [
        [
            artifactId: 'sun_war', 
            classifier: '', 
            file: 'target/test-sun_war.war', 
            type: 'war'
        ]
    ], 
    credentialsId: 'NEXUS_KEY', 
    groupId: 'msingh977', 
    nexusUrl: 'http://54.208.146.184:8044/', 
    nexusVersion: 'nexus3', 
    protocol: 'http', 
    repository: 'msingh977-release', 
    version: '1.0.0'
     }
}

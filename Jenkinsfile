import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles)
    if (p['publishMethod'] == 'FTP')
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
}

node {
  withEnv(['AZURE_SUBSCRIPTION_ID=d105f22e-7313-4267-b3b2-572544fc707a',
        'AZURE_TENANT_ID=4e31fdef-8b04-47cb-a9f6-8c0b2ad482e1']) {
    stage('init') {
      checkout scm
    }
  
    stage('build') {
      sh 'mvn clean package'
    }
  
    stage('deploy') {
      def resourceGroup = 'GitHub-DevOps'
      def webAppName = 'javasamplewebapp'
      // login Azure
      withCredentials([usernamePassword(credentialsId: '070c6048-e25d-4668-9dd3-ed761656bdfb', passwordVariable: '4boeOM2v1k7w3uK6KlsaINQ.f-m5f-h0Pa', usernameVariable: '98e5ba44-9ae2-45fb-bf3d-72acd3abe3e8')]) {
       sh '''
          az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET -t $AZURE_TENANT_ID
          az account set -s $AZURE_SUBSCRIPTION_ID
        '''
      }
      // get publish settings
      def pubProfilesJson = sh script: "az webapp deployment list-publishing-profiles -g $resourceGroup -n $webAppName", returnStdout: true
      def ftpProfile = getFtpPublishProfile pubProfilesJson
      // upload package
      sh "curl -T target/calculator-1.0.war $ftpProfile.url/webapps/ROOT.war -u '$ftpProfile.username:$ftpProfile.password'"
      // log out
      sh 'az logout'
    }
  }
}

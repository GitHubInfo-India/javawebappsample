import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles)
    if (p['publishMethod'] == 'FTP')
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
}

node {
  withEnv(['AZURE_SUBSCRIPTION_ID=d105f22e-7313-4267-b3b2-572544fc707a',
        'AZURE_TENANT_ID=4e31fdef-8b04-47cb-a9f6-8c0b2ad482e1',
        'AZURE_CLIENT_ID=53aab92e-8a00-4f0b-b1d7-2c347111c463',
        'AZURE_CLIENT_SECRET=fT6kWVYF.Ftcf9HH77w09tsY~L1HVynGNf']) {
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
      withCredentials([usernamePassword(credentialsId: '9baded81-cae5-4e02-abb5-28fc69ef5519', passwordVariable: 'fT6kWVYF.Ftcf9HH77w09tsY~L1HVynGNf', usernameVariable: '53aab92e-8a00-4f0b-b1d7-2c347111c463')]) {
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

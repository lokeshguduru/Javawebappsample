import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles)
    if (p['publishMethod'] == 'FTP')
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
}

node {
  withEnv(['AZURE_SUBSCRIPTION_ID=4ab87186-65ae-4fea-b37a-877edcb1d208',
        'AZURE_TENANT_ID=667d8bc2-ac29-4294-8e84-40f3c4defef4']) {
    stage('init') {
      checkout scm
    }
  
    stage('build') {
      sh 'mvn clean package'
    }
  
    stage('deploy') {
      def resourceGroup = 'loki'
      def webAppName = 'samplejava'
      // login Azure
      ///withCredentials([usernamePassword(credentialsId: 'eac082ad-4ec7-407d-add3-900a0f5381be', passwordVariable: 'AZURE_CLIENT_SECRET', usernameVariable: 'AZURE_CLIENT_ID')]) {
       sh '''
          az login --service-principal -u d1d3771d-c722-444c-8ccb-4d9decdaf5c0 -p ftl8Q~-eMsrFEDpMZpR9O.LYMvIIEdNsQVjvGbk4 -t 667d8bc2-ac29-4294-8e84-40f3c4defef4
          az account set -s $AZURE_SUBSCRIPTION_ID
        '''
      ///}
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
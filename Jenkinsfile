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
      def resourceGroup = 'Loki'
      def webAppName = 'sampleberm'
      // login Azure
  	 // withCredentials([azureServicePrincipal(credentialsId: 'd1d3771d-c722-444c-8ccb-4d9decdaf5c0', passwordVariable: '5263fc3b-244f-4869-937e-ed3a3936bf05', usernameVariable: 'AZURE_CLIENT_ID', tenantIdVariable: 'AZURE_TENANT_ID')]) {
   	  sh '''
      		az login --service-principal -u d1d3771d-c722-444c-8ccb-4d9decdaf5c0 -p 5263fc3b-244f-4869-937e-ed3a3936bf05 -t $AZURE_TENANT_ID
      	  	az account set -s 667d8bc2-ac29-4294-8e84-40f3c4defef4
    	   '''
    //  }
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

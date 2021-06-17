import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles)
    if (p['publishMethod'] == 'FTP')
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
}

node {
  withEnv(['AZURE_SUBSCRIPTION_ID=da74e3ed-9c9a-4605-8fae-10f3492c0f5c',
        'AZURE_TENANT_ID=9b415834-803a-4da0-afdc-fe6b1d52d649']) {
    stage('init') {
      checkout scm
    }
  
    stage('build') {
      sh 'mvn clean package'
    }
  
    stage('deploy') {
      def resourceGroup = 'UPS-WMP-DEV_RG'
      def webAppName = 'wechatdevstatic'
      // login Azure
      withCredentials([usernamePassword(credentialsId: 'UPS-WMP-DEV-AKSSP-20210122150559', passwordVariable: '7wAl2H2.1PhShUWB96Y_j-2q1S8.xr9k2n', usernameVariable: 'ca2523dd-04e0-4752-8f8d-bb55882f334c')]) {
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

import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles)
    if (p['publishMethod'] == 'FTP')
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
}

node {
  withEnv(['AZURE_SUBSCRIPTION_ID=ddc30a6d-6005-4bd6-8d62-19e0f856f6a3',
           'AZURE_TENANT_ID=30983c32-cacd-4133-851d-394735cedebe']) {
    
    stage('init') {
      checkout scm
    }

    stage('build') {
      sh 'mvn clean package -DskipTests'
    }

    stage('deploy') {
      def resourceGroup = 'jenkins-get-started-rg'
      def webAppName = 'yuzhang2025app'

      withCredentials([usernamePassword(credentialsId: 'AzureServicePrincipal', 
                                        passwordVariable: 'AZURE_CLIENT_SECRET', 
                                        usernameVariable: 'AZURE_CLIENT_ID')]) {
        sh '''
          az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET -t $AZURE_TENANT_ID
          az account set -s $AZURE_SUBSCRIPTION_ID
        '''
      }

      def pubProfilesJson = sh script: "az webapp deployment list-publishing-profiles -g $resourceGroup -n $webAppName", returnStdout: true
      def ftpProfile = getFtpPublishProfile pubProfilesJson

      sh "curl -T target/calculator-1.0.war $ftpProfile.url/webapps/ROOT.war -u '$ftpProfile.username:$ftpProfile.password'"

      sh 'az logout'
    }
  }
}

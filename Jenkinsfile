pipeline {
agent { docker { image 'yourorg/sfdx-agent:latest' } }
options { ansiColor('xterm'); timestamps() }
stages {
stage('Checkout') {
steps { checkout scm }
}
stage('Authenticate') {
steps {
withCredentials([file(credentialsId: 'sfdc_jwt_key', variable: 'JWT_KEY'),
string(credentialsId: 'sfdc_client_id', variable: 'SF_CLIENT_ID'),
string(credentialsId: 'sfdc_username', variable: 'SF_USERNAME'),
string(credentialsId: 'sfdc_instance_url', variable: 'SF_INSTANCE_URL')]) {
sh '''
set -e
echo "Authenticating to Salesforce..."
sfdx auth:jwt:grant --clientid $SF_CLIENT_ID --jwtkeyfile $JWT_KEY --username $SF_USERNAME '''
}
}
}
stage('Validate (checkonly)') {
steps {
sh 'sfdx force:source:deploy -x manifest/package.xml --checkonly --testlevel RunLocalTests --}
}
stage('Deploy to Sandbox') {
steps {
sh 'sfdx force:source:deploy -x manifest/package.xml --testlevel RunLocalTests --wait 20'
}
}
stage('Post-Tests & Reports') {
steps {
sh 'sfdx force:apex:test:run --resultformat human --wait 20 || true'
}
}
}
post {
success { echo 'Deployment succeeded' }
failure {
echo 'Deployment failed, creating backup...'
sh 'sfdx force:mdapi:retrieve -k manifest/package.xml -r ./backup || true'
}
}
}

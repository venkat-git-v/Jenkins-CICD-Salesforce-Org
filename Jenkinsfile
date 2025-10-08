pipeline {
    agent any

    environment {
        SF_CONSUMER_KEY = credentials('SALESFORCE_CONSUMER_KEY')
        SF_USERNAME = 'your-automation-user@yourdomain.com'
        SF_LOGIN_URL = 'https://login.salesforce.com'
        JWT_KEY_FILE = credentials('SALESFORCE_SERVER_KEY') // Jenkins file credential (private key)
        SFDX_CLI = '/usr/local/bin/sfdx' // Adjust path as needed
    }
  stages {
        stage('Run Salesforce CLI') {
            steps {
                bat 'sfdx --version' // or any other sfdx command
            }
        }
    stages {
        stage('Checkout Source') {
            steps {
                checkout scm
            }
        }

        stage('Authorize Salesforce Org via JWT') {
            steps {
                bat """
                ${SFDX_CLI} force:auth:jwt:grant \
                  --clientid ${SF_CONSUMER_KEY} \
                  --jwtkeyfile ${JWT_KEY_FILE} \
                  --username ${SF_USERNAME} \
                  --instanceurl ${SF_LOGIN_URL} \
                  --setdefaultusername
                """
            }
        }

        stage('Create Scratch Org') {
            steps {
                bat "${SFDX_CLI} force:org:create -f config/project-scratch-def.json -a test_scratch -s"
            }
        }

        stage('Push Source to Scratch Org') {
            steps {
                bat "${SFDX_CLI} force:source:push -u test_scratch"
            }
        }

        stage('Run Apex Tests') {
            steps {
                bat "${SFDX_CLI} force:apex:test:run -u test_scratch --wait 10"
            }
        }

        stage('Deploy to Staging/Production') {
            steps {
                input "Proceed to deploy to destination org?"
                bat "${SFDX_CLI} force:source:deploy -u DEST_ORG_ALIAS -p force-app/"
            }
        }
    }
  }
    post {
        always {
            bat "${SFDX_CLI} force:org:delete -u test_scratch --noprompt"
        }
    }
}

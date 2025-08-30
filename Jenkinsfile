pipeline {
    agent any
    
    parameters {
        string(name: 'PACKAGE_VERSION', defaultValue: '04tXXXXXXXXXXXXXXX', description: 'Salesforce Package Version to deploy')
        choice(name: 'TEST_LEVEL', choices: ['NoTestRun', 'RunSpecifiedTests', 'RunLocalTests', 'RunAllTestsInOrg'], description: 'Test Level for deployment')
        booleanParam(name: 'DEPLOY_TO_UAT', defaultValue: true, description: 'Whether to deploy to UAT after QA')
    }
    
    environment {
        SERVER_KEY_CREDENTALS_ID = "${env.SERVER_KEY_CREDENTALS_ID}"
        SF_INSTANCE_URL = "${env.SF_INSTANCE_URL}"
        
        SF_CONSUMER_KEY = "${env.SF_Dev_CONSUMER_KEY}"
        SF_USERNAME = "${env.SF_Dev_Int_USERNAME}"
        
        SF_QA_CONSUMER_KEY = "${env.SF_QA_CONSUMER_KEY}"
        SF_QA_USERNAME = "${env.SF_QA_Int_USERNAME}"
        
        SF_UAT_CONSUMER_KEY = "${env.SF_UAT_CONSUMER_KEY}"
        SF_UAT_USERNAME = "${env.SF_UAT_Int_USERNAME}"

        // SLACK_CHANNEL = '#your-slack-channel'   // Change to your channel
        // SLACK_CREDENTIAL_ID = 'slack-token-id'  // Jenkins Credential ID for Slack token
        EMAIL_RECIPIENTS = 'team@example.com'   // Change to your mailing list
    }
    
    stages {
        stage('Verify Salesforce CLI') {
            steps {
                script {
                    def sfVersion = sh(script: 'sf --version', returnStdout: true).trim()
                    echo "Salesforce CLI version: ${sfVersion}"
                }
            }
        }
        
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Deploy to Dev') {
            steps {
                withCredentials([file(credentialsId: SERVER_KEY_CREDENTALS_ID, variable: 'SERVER_KEY')]) {
                    script {
                        echo "Deploying to Dev org: ${SF_USERNAME}"
                        sh """
                            sf auth:jwt:grant --clientid ${SF_CONSUMER_KEY} --jwtkeyfile \$SERVER_KEY --username ${SF_USERNAME} --instanceurl ${SF_INSTANCE_URL}
                            sf package version promote -p ${params.PACKAGE_VERSION} --noprompt --targetusername ${SF_USERNAME}
                            sf run test --targetusername ${SF_USERNAME} --testlevel ${params.TEST_LEVEL}
                        """
                    }
                }
            }
        }

        stage('Deploy to QA') {
            steps {
                withCredentials([file(credentialsId: SERVER_KEY_CREDENTALS_ID, variable: 'SERVER_KEY')]) {
                    script {
                        echo "Deploying to QA org: ${SF_QA_USERNAME}"
                        sh """
                            sf auth:jwt:grant --clientid ${SF_QA_CONSUMER_KEY} --jwtkeyfile \$SERVER_KEY --username ${SF_QA_USERNAME} --instanceurl ${SF_INSTANCE_URL}
                            sf package version promote -p ${params.PACKAGE_VERSION} --noprompt --targetusername ${SF_QA_USERNAME}
                            sf run test --targetusername ${SF_QA_USERNAME} --testlevel ${params.TEST_LEVEL}
                        """
                    }
                }
            }
        }

        stage('Deploy to UAT') {
            when {
                expression { return params.DEPLOY_TO_UAT }
            }
            steps {
                withCredentials([file(credentialsId: SERVER_KEY_CREDENTALS_ID, variable: 'SERVER_KEY')]) {
                    script {
                        echo "Deploying to UAT org: ${SF_UAT_USERNAME}"
                        sh """
                            sf auth:jwt:grant --clientid ${SF_UAT_CONSUMER_KEY} --jwtkeyfile \$SERVER_KEY --username ${SF_UAT_USERNAME} --instanceurl ${SF_INSTANCE_URL}
                            sf package version promote -p ${params.PACKAGE_VERSION} --noprompt --targetusername ${SF_UAT_USERNAME}
                            sf run test --targetusername ${SF_UAT_USERNAME} --testlevel ${params.TEST_LEVEL}
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            script {
                echo 'Deployment pipeline completed successfully!'
                /*
                slackSend(
                    channel: env.SLACK_CHANNEL,
                    color: 'good',
                    message: "Salesforce Deployment SUCCESS: Package ${params.PACKAGE_VERSION} deployed to Dev, QA${ params.DEPLOY_TO_UAT ? ', and UAT' : '' }."
                )
                */
                mail(
                    to: env.EMAIL_RECIPIENTS,
                    subject: "SUCCESS: Salesforce Deployment ${params.PACKAGE_VERSION}",
                    body: "The Salesforce package version ${params.PACKAGE_VERSION} has been deployed successfully to Dev, QA${ params.DEPLOY_TO_UAT ? ', and UAT' : '' } orgs."
                )
            }
        }
        failure {
            script {
                echo 'Deployment pipeline failed.'
                /*
                slackSend(
                    channel: env.SLACK_CHANNEL,
                    color: 'danger',
                    message: "Salesforce Deployment FAILURE: Check Jenkins build ${env.BUILD_URL}."
                )
                */
                mail(
                    to: env.EMAIL_RECIPIENTS,
                    subject: "FAILURE: Salesforce Deployment",
                    body: "The Salesforce deployment failed. Check Jenkins build: ${env.BUILD_URL}"
                )
            }
        }
    }
}

/*
 * This Jenkinsfile automates a Salesforce CI/CD pipeline.
 * It is triggered by a push to the 'dev' branch and deploys
 * a delta package to the QA org.
 */
pipeline {
    agent any

    environment {
        // As per request, using a hardcoded PATH and a pre-defined credential ID.
        PATH = "/Users/kburugu/.nvm/versions/node/v20.19.0/bin:$PATH"

        // Credentials for the private key.
        SERVER_KEY_CREDENTIALS_ID = "${env.SERVER_KEY_CREDENTIALS_ID}"

        // Set email recipients to a specific email address.
        EMAIL_RECIPIENTS = "chanrdra@gmail.com"
        
        // Dev Org credentials (source of truth for the pipeline)
        SF_CONSUMER_KEY = "${env.SF_DEV_CONSUMER_KEY}"
        SF_USERNAME = "${env.SF_DEV_INT_USERNAME}"
        SF_INSTANCE_URL = "${env.SF_DEV_INSTANCE_URL}"
        
        // QA Org credentials (target for the deployment)
        SF_QA_CONSUMER_KEY = "${env.SF_QA_CONSUMER_KEY}"
        SF_QA_USERNAME = "${env.SF_QA_INT_USERNAME}"
        SF_QA_INSTANCE_URL = "${env.SF_QA_INSTANCE_URL}"
        
        // The test level to run
        TEST_LEVEL = 'RunLocalTests'
    }

    stages {
        stage('Checkout') {
            steps {
                // The `checkout scm` step automatically clones the repository based on the job configuration.
                checkout scm
            }
        }

        stage('Deploy to QA') {
            // This stage will now only run if the branch is 'dev'
            when {
                branch 'dev'
            }
            steps {
                script {
                    // A helper function to perform authentication and deployment.
                    def deployToOrg(orgName, consumerKey, username, instanceUrl, deltaPackagePath) {
                        withCredentials([file(credentialsId: SERVER_KEY_CREDENTIALS_ID, variable: 'SERVER_KEY')]) {
                            sh """#!/bin/bash -e
                                set +x
                                echo 'Authenticating with JWT...'
                                sf auth:jwt:grant --clientid ${consumerKey} --jwt-key-file "\$SERVER_KEY" --username ${username} --instanceurl ${instanceUrl}
                                set -x
                                
                                echo 'Deploying delta package to Salesforce Org...'
                                sf project deploy start --target-org ${username} --source-dir ${deltaPackagePath} --test-level ${TEST_LEVEL} --wait 10
                                
                                echo '✅ Deployment to ${orgName} completed successfully!'
                            """
                        }
                    }
                    // Create a delta package based on the last two commits
                    sh 'sfdx-git-delta --from HEAD^1 --to HEAD --output ./.delta-package'
                    
                    // Call the reusable function to perform the deployment
                    deployToOrg('QA', SF_QA_CONSUMER_KEY, SF_QA_USERNAME, SF_QA_INSTANCE_URL, '.delta-package')
                }
            }
        }
    }

    // Post-build actions for success and failure.
    post {
        success {
            echo '✅ Deployment pipeline completed successfully!'
            mail to: "${EMAIL_RECIPIENTS}",
                 subject: "SUCCESS: Jenkins Deployment Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                 body: """Hello Team,

The deployment to QA completed successfully.

Job: ${env.JOB_NAME}
Build Number: ${env.BUILD_NUMBER}
Build URL: ${env.BUILD_URL}

Best,
Jenkins"""
        }
        failure {
            echo '❌ Deployment pipeline failed. Check logs.'
            mail to: "${EMAIL_RECIPIENTS}",
                 subject: "FAILURE: Jenkins Deployment Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                 body: """Hello Team,

The deployment to QA has failed.

Job: ${env.JOB_NAME}
Build Number: ${env.BUILD_NUMBER}
Build URL: ${env.BUILD_URL}

Please check the logs and investigate.

Best,
Jenkins"""
        }
    }
}

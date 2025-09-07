/*
 * This Jenkinsfile automates a Salesforce CI/CD pipeline.
 * It is triggered by a push to the 'dev' branch and deploys
 * all the code to the Dev org, then to the QA org.
 */
pipeline {
    agent any

    environment {
        // As per request, using a hardcoded PATH and a pre-defined credential ID.
        PATH = "/Users/kburugu/.nvm/versions/node/v20.19.0/bin:$PATH"

        // Credentials for the private key.
        SERVER_KEY_CREDENTIALS_ID = "${env.SERVER_KEY_CREDENTALS_ID}"

        SF_INSTANCE_URL = "${env.SF_INSTANCE_URL}"

        // Set email recipients to a specific email address.
        EMAIL_RECIPIENTS = "chanrdra@gmail.com"
        
        // Dev Org credentials (target for the deployment)
        SF_DEV_CONSUMER_KEY = "${env.SF_DEV_CONSUMER_KEY}"
        SF_DEV_USERNAME = "${env.SF_DEV_INT_USERNAME}"
      
        
        // QA Org credentials (target for the deployment)
        SF_QA_CONSUMER_KEY = "${env.SF_QA_CONSUMER_KEY}"
        SF_QA_USERNAME = "${env.SF_QA_INT_USERNAME}"
     
        
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
        
        stage('Deploy to Dev') {
            // This stage will run for every push to the dev branch.
            checkout dev
            when {
                branch 'dev'
            }
            steps {
                script {
                    // A helper function to perform authentication and deployment to Dev.
                
                        withCredentials([file(credentialsId: SERVER_KEY_CREDENTALS_ID, variable: 'SERVER_KEY')]) {
                            sh """#!/bin/bash -e
                                set +x
                                echo 'Authenticating with JWT...'
                                sf auth:jwt:grant --clientid ${SF_DEV_CONSUMER_KEY} --jwt-key-file "\$SERVER_KEY" --username ${SF_DEV_USERNAME} --instanceurl ${SF_INSTANCE_URL}
                                set -x
                                
                                echo 'Deploying all changes from repository to Dev Org...'
                                // Deploy all source code to the Dev Org
                                sf project deploy start --target-org ${username} --source-dir force-app/main/default --wait 10 --test-level ${TEST_LEVEL}
                                  echo 'Authorized Successfully'
                                echo '✅ Deployment to Dev completed successfully!'
                            """
                        }
                    
                    
                    // Call the function to deploy to the Dev Org
                  //  deployToDevOrg(SF_DEV_CONSUMER_KEY, SF_DEV_USERNAME, SF_DEV_INSTANCE_URL)
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

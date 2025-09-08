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
       // SF_DEV_Deploy_USERNAME = "${env.SF_Dev_Admin_ID}"
      
        
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
        
        stage('Deploy to Dev Started') {
            // This stage will run for every push to the dev branch.
           // when {
           //     branch '*/dev'
          //  }
            steps {
                script {
                    // A helper function to perform authentication and deployment to Dev.
                
                        withCredentials([file(credentialsId: SERVER_KEY_CREDENTALS_ID, variable: 'SERVER_KEY')]) {
                            sh """#!/bin/bash -e
                                set +x
                                # echo 'Authenticating with JWT...for Dev Org'
                                 # sf auth:jwt:grant --clientid ${SF_DEV_CONSUMER_KEY} --jwt-key-file "${SERVER_KEY}" --username ${SF_DEV_USERNAME} --instanceurl ${SF_INSTANCE_URL}
                                  echo 'Authenticating with JWT...for QA Org'
                                 sf auth:jwt:grant --clientid ${SF_QA_CONSUMER_KEY} --jwt-key-file "${SERVER_KEY}" --username ${SF_QA_USERNAME} --instanceurl ${SF_INSTANCE_URL}
                               
                                set -x
                                
                                  # Find all changed Apex classes since the last commit
                            CHANGED_APEX_FILES=\$(git diff --name-only HEAD~1 | grep 'force-app/main/default/classes/.*.cls')

                            # Extract just the class names and format for the --metadata flag
                            APEX_METADATA=\$(echo "\$CHANGED_APEX_FILES" | sed 's/force-app\\/main\\/default\\/classes\\/\\(.*\\).cls/ApexClass:\\1/g' | tr '\\n' ',')
                            
                             # Remove the trailing comma
                            APEX_METADATA=\${APEX_METADATA%,}

                            # If no Apex files were changed, exit gracefully
                            if [ -z "\$CHANGED_APEX_FILES" ]; then
                                echo 'No Apex classes changed, skipping deployment.'
                                exit 0
                            fi

                               echo "Deploying the following Apex classes: \${APEX_METADATA}"
                            # Deploy only the changed Apex classes to the Dev Org
                            
                            #   sf project deploy start --target-org ${SF_QA_USERNAME} --metadata "\${APEX_METADATA}" --wait 10 --test-level ${TEST_LEVEL}
                            echo 'New Deploy Apex classes changed, skipping deployment.'
                            #  sf deploy metadata --metadata "\${APEX_METADATA}" --target-org ${SF_QA_USERNAME}
                            sf project deploy start --metadata ApexClass:DemoDev --ignore-conflicts
                            echo 'Authorized Successfully and Checking'
                            echo '✅ Deployment to Dev completed successfully!'
                            """
                        }
                    
                    // 
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

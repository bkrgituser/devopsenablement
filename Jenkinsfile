pipeline {
    agent any
    
    environment {
        // Global from Manage Jenkins > Configure System > Global properties
        SERVER_KEY_CREDENTALS_ID = 'SF_PRIVATE_KEY_CREDENTIAL_ID'
        SF_INSTANCE_URL = 'https://login.salesforce.com'  // or your My Domain URL
        
        // Dev org vars
        SF_CONSUMER_KEY = "${env.SF_Dev_CONSUMER_KEY}"
        SF_USERNAME = "${env.SF_Dev_Int_USERNAME}"
        
        // QA org vars (will override in QA stage)
        SF_QA_CONSUMER_KEY = "${env.SF_QA_CONSUMER_KEY}"
        SF_QA_USERNAME = "${env.SF_QA_Int_USERNAME}"
        
        // UAT org vars (will override in UAT stage)
        SF_UAT_CONSUMER_KEY = "${env.SF_UAT_CONSUMER_KEY}"
        SF_UAT_USERNAME = "${env.SF_UAT_Int_USERNAME}"

        PACKAGE_NAME = 'My Package'          // Set your package name
        PACKAGE_VERSION = '04tXXXXXXXXXXXXXXX' // Starting package version - update in script if needed
        TEST_LEVEL = 'RunLocalTests'         // Adjust as needed
    }
    
    stages {
        stage('Checkout') {
            steps {
                // Checkout your Git repo
                checkout scm
            }
        }

        stage('Deploy to Dev') {
            steps {
                withCredentials([file(credentialsId: SERVER_KEY_CREDENTALS_ID, variable: 'SERVER_KEY')]) {
                    script {
                        echo "Authenticating to Dev org: ${SF_USERNAME}"
                        sh """
                            sf auth:jwt:grant --clientid ${SF_CONSUMER_KEY} --jwtkeyfile \$SERVER_KEY --username ${SF_USERNAME} --instanceurl ${SF_INSTANCE_URL}
                            sf package version promote -p ${PACKAGE_VERSION} --noprompt --targetusername ${SF_USERNAME}
                            sf run test --targetusername ${SF_USERNAME} --testlevel ${TEST_LEVEL}
                        """
                    }
                }
            }
        }

        stage('Deploy to QA') {
            steps {
                withCredentials([file(credentialsId: SERVER_KEY_CREDENTALS_ID, variable: 'SERVER_KEY')]) {
                    script {
                        echo "Authenticating to QA org: ${SF_QA_USERNAME}"
                        sh """
                            sf auth:jwt:grant --clientid ${SF_QA_CONSUMER_KEY} --jwtkeyfile \$SERVER_KEY --username ${SF_QA_USERNAME} --instanceurl ${SF_INSTANCE_URL}
                            sf package version promote -p ${PACKAGE_VERSION} --noprompt --targetusername ${SF_QA_USERNAME}
                            sf run test --targetusername ${SF_QA_USERNAME} --testlevel ${TEST_LEVEL}
                        """
                    }
                }
            }
        }

        stage('Deploy to UAT') {
            steps {
                withCredentials([file(credentialsId: SERVER_KEY_CREDENTALS_ID, variable: 'SERVER_KEY')]) {
                    script {
                        echo "Authenticating to UAT org: ${SF_UAT_USERNAME}"
                        sh """
                            sf auth:jwt:grant --clientid ${SF_UAT_CONSUMER_KEY} --jwtkeyfile \$SERVER_KEY --username ${SF_UAT_USERNAME} --instanceurl ${SF_INSTANCE_URL}
                            sf package version promote -p ${PACKAGE_VERSION} --noprompt --targetusername ${SF_UAT_USERNAME}
                            sf run test --targetusername ${SF_UAT_USERNAME} --testlevel ${TEST_LEVEL}
                        """
                    }
                }
            }
        }
    }
    
    post {
        success {
            echo 'Deployment pipeline completed successfully!'
        }
        failure {
            echo 'Deployment pipeline failed. Please check the logs.'
        }
    }
}

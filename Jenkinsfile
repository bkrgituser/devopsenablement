pipeline {
    agent any

    environment {
        
         PATH = "/Users/kburugu/.nvm/versions/node/v20.19.0/bin:$PATH"
        SERVER_KEY_CREDENTALS_ID = "${env.SERVER_KEY_CREDENTALS_ID}"

        SF_INSTANCE_URL = "${env.SF_INSTANCE_URL}"
        
        // Dev Org
        SF_CONSUMER_KEY       = "${env.SF_Dev_CONSUMER_KEY}"
        SF_USERNAME           = "${env.SF_Dev_Int_USERNAME}"

        // QA Org
        SF_QA_CONSUMER_KEY    = "${env.SF_QA_CONSUMER_KEY}"
        SF_QA_USERNAME        = "${env.SF_QA_Int_USERNAME}"

        // UAT Org
        SF_UAT_CONSUMER_KEY   = "${env.SF_UAT_CONSUMER_KEY}"
        SF_UAT_USERNAME       = "${env.SF_UAT_Int_USERNAME}"

        PACKAGE_NAME    = 'My Package'
        PACKAGE_VERSION = '04tXXXXXXXXXXXXXXX' // Replace with your actual package version
        TEST_LEVEL      = 'RunLocalTests'

        EMAIL_RECIPIENTS = 'chanrdra@gmail.com' // Add your email recipients here or via Jenkins env config
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Deploy to Dev') {
            steps {
                withCredentials([file(credentialsId: SERVER_KEY_CREDENTALS_ID, variable: 'SERVER_KEY')]) {
                    sh """
                        sf auth:jwt:grant --clientid ${SF_CONSUMER_KEY} --jwtkeyfile \$SERVER_KEY --username ${SF_USERNAME} --instanceurl ${SF_INSTANCE_URL}
                       
                        sf run test --targetusername ${SF_USERNAME} --testlevel ${TEST_LEVEL}
                        echo '✅ Deployment to Dev completed successfully!'
                    """
                }
            }
        }

        stage('Deploy to QA') {
            steps {
                withCredentials([file(credentialsId: SERVER_KEY_CREDENTALS_ID, variable: 'SERVER_KEY')]) {
                    sh """
                        sf auth:jwt:grant --clientid ${SF_QA_CONSUMER_KEY} --jwtkeyfile \$SERVER_KEY --username ${SF_QA_USERNAME} --instanceurl ${SF_INSTANCE_URL}
                       
                        sf run test --targetusername ${SF_QA_USERNAME} --testlevel ${TEST_LEVEL}
                        echo '✅ Deployment to QA completed successfully!'
                    """
                }
            }
        }

        stage('Deploy to UAT') {
            steps {
                withCredentials([file(credentialsId: SERVER_KEY_CREDENTALS_ID, variable: 'SERVER_KEY')]) {
                    sh """
                        sf auth:jwt:grant --clientid ${SF_UAT_CONSUMER_KEY} --jwtkeyfile \$SERVER_KEY --username ${SF_UAT_USERNAME} --instanceurl ${SF_INSTANCE_URL}
                        
                        sf run test --targetusername ${SF_UAT_USERNAME} --testlevel ${TEST_LEVEL}
                        echo '✅ Deployment to UAT completed successfully!'
                    """
                }
            }
        }
    }

    post {
        success {
            echo '✅ Deployment pipeline completed successfully!'
            mail to: "${EMAIL_RECIPIENTS}",
                 subject: "SUCCESS: Jenkins Deployment Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                 body: """Hello Team,

The deployment pipeline completed successfully.

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

The deployment pipeline has failed.

Job: ${env.JOB_NAME}
Build Number: ${env.BUILD_NUMBER}
Build URL: ${env.BUILD_URL}

Please check the logs and investigate.

Best,
Jenkins"""
        }
    }
}

pipeline {
    agent any
    
    environment {
        BUILD_NUMBER = "${env.BUILD_NUMBER}"
        RUN_ARTIFACT_DIR = "tests/${env.BUILD_NUMBER}"
    }
    
    parameters {
        string(defaultValue: '', description: 'Salesforce Username', name: 'SFDC_USERNAME')
    }

    stages {
        stage('Initialize') {
            steps {
                script {
                    def HUB_ORG = env.HUB_ORG_DH
                    def SFDC_HOST = env.SFDC_HOST_DH
                    def JWT_KEY_CRED_ID = env.JWT_CRED_ID_DH
                    def CONNECTED_APP_CONSUMER_KEY = env.CONNECTED_APP_CONSUMER_KEY_DH
                    
                    echo 'Environment variables:'
                    echo "BUILD_NUMBER: ${BUILD_NUMBER}"
                    echo "HUB_ORG: ${HUB_ORG}"
                    echo "SFDC_HOST: ${SFDC_HOST}"
                    echo "JWT_KEY_CRED_ID: ${JWT_KEY_CRED_ID}"
                    echo "CONNECTED_APP_CONSUMER_KEY: ${CONNECTED_APP_CONSUMER_KEY}"
                    
                    toolbelt = tool 'toolbelt'
                }
            }
        }
        
        stage('Checkout Source') {
            steps {
                checkout scm
            }
        }
        
        stage('Deploy Code') {
            steps {
                script {
                    withCredentials([file(credentialsId: JWT_KEY_CRED_ID, variable: 'jwt_key_file')]) {
                        if (isUnix()) {
                            rc = sh returnStatus: true, script: "${toolbelt} force:auth:jwt:grant --clientid ${CONNECTED_APP_CONSUMER_KEY} --username ${HUB_ORG} --jwtkeyfile ${jwt_key_file} --setdefaultdevhubusername --instanceurl ${SFDC_HOST}"
                        } else {
                            rc = bat returnStatus: true, script: "\"${toolbelt}\" force:auth:jwt:grant --clientid ${CONNECTED_APP_CONSUMER_KEY} --username ${HUB_ORG} --jwtkeyfile \"${jwt_key_file}\" --setdefaultdevhubusername --instanceurl ${SFDC_HOST}"
                        }
                        if (rc != 0) { error 'Hub org authorization failed' }

                        echo "Authorization successful with return code: ${rc}"

                        if (isUnix()) {
                            rmsg = sh returnStdout: true, script: "${toolbelt} force:source:deploy --manifest manifest/package.xml -u ${HUB_ORG}"
                        } else {
                            rmsg = bat returnStdout: true, script: "\"${toolbelt}\" force:source:deploy --manifest manifest/package.xml -u ${HUB_ORG}"
                        }

                        echo "Deployment result:"
                        echo rmsg
                        echo 'Hello from a Job DSL script!'
                    }
                }
            }
        }
    }
}

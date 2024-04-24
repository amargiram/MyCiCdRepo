#!groovy
import groovy.json.JsonSlurperClassic

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
                    
                    println 'KEY IS'
                    println JWT_KEY_CRED_ID
                    println HUB_ORG
                    println SFDC_HOST
                    println CONNECTED_APP_CONSUMER_KEY
                    
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
                withCredentials([file(credentialsId: "${JWT_KEY_CRED_ID}", variable: 'jwt_key_file')]) {
                    script {
                        if (isUnix()) {
                            rc = sh returnStatus: true, script: "${toolbelt} force:auth:jwt:grant --clientid ${CONNECTED_APP_CONSUMER_KEY} --username ${HUB_ORG} --jwtkeyfile ${jwt_key_file} --setdefaultdevhubusername --instanceurl ${SFDC_HOST}"
                        } else {
                            rc = bat returnStatus: true, script: "\"${toolbelt}\" force:auth:jwt:grant --clientid ${CONNECTED_APP_CONSUMER_KEY} --username ${HUB_ORG} --jwtkeyfile \"${jwt_key_file}\" --setdefaultdevhubusername --instanceurl ${SFDC_HOST}"
                        }
                        if (rc != 0) { error 'Hub org authorization failed' }

                        println rc

                        if (isUnix()) {
                            rmsg = sh returnStdout: true, script: "${toolbelt} force:source:deploy --manifest manifest/package.xml -u ${HUB_ORG}"
                        } else {
                            rmsg = bat returnStdout: true, script: "\"${toolbelt}\" force:source:deploy --manifest manifest/package.xml -u ${HUB_ORG}"
                        }

                        printf rmsg
                        println('Hello from a Job DSL script!')
                        println(rmsg)
                    }
                }
            }
        }
    }
}

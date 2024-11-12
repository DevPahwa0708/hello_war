#!/usr/bin/env groovy
import hudson.model.*
import hudson.EnvVars
import groovy.json.JsonSlurperClassic
import groovy.json.JsonBuilder
import groovy.json.JsonOutput
import java.net.URL

pipeline {
    agent any
    tools {
        maven 'maven'
        jdk 'java-1.8'
    }
    parameters {
        // A choice parameter to select which Tomcat instance to deploy to
        choice(name: 'TOMCAT_INSTANCE', choices: ['tomcat1', 'tomcat3', 'tomcat4', 'tomcat5', 'tomcat6', 'tomcat7', 'tomcat4_local_server', 'tomcat-cscac', 'tomcat-wac', 'tomcat_wat', 'tomcat_wet'], 
               description: 'Select the Tomcat instance to deploy to')
    }
    environment {
        SERVER_USER = 'ddev'           // Replace with your server's username
        SERVER_HOST = '192.168.200.70'     // Replace with your server's hostname or IP
        SERVER_PORT = '5505'
        SERVER_PATH = '/home/ddev/jenkins/' // Replace with your destination directory on the server
       // SERVER_PASSWORD = credentials('wb-uat') // Use Jenkins credentials for the server passwordy
    }
    stages {
        stage('Build') {
            steps {
                echo "Creating New Build"
                sh 'mvn clean install'
            }
        }
        stage('Copy WAR file to Server') {
            steps {
                script {
                    // Find the WAR file in the target directory
                    //def warFile = sh(script: "find target -name '*.war'", returnStdout: true).trim()
                    def warFile = sh(script: "find target -maxdepth 1 -name '*.war'", returnStdout: true).trim()
                    
                    if (warFile) {
                        echo "Found WAR file: ${warFile}"

                        // Use withCredentials to securely inject the password into the pipeline
                        withCredentials([string(credentialsId: 'pass_wb_uat', variable: 'SERVER_PASSWORD')]) {
                            // Use sshpass to provide the password and copy the WAR file via SCP
                            sh """
                                sshpass -p \$SERVER_PASSWORD scp -o StrictHostKeyChecking=no -rP ${SERVER_PORT} ${warFile} ${SERVER_USER}@${SERVER_HOST}:${SERVER_PATH}
                            """
                            // Determine the correct deploy.sh path based on the TOMCAT_INSTANCE parameter
                            def deployScriptPath = ''
                            switch (params.TOMCAT_INSTANCE) {
                                case 'tomcat1':
                                    deployScriptPath = "${SERVER_PATH}/tomcat1/deploy.sh"
                                    break
                                case 'tomcat3':
                                    deployScriptPath = "${SERVER_PATH}/tomcat3/deploy.sh"
                                    break
                                case 'tomcat4':
                                    deployScriptPath = "${SERVER_PATH}/tomcat4/deploy.sh"
                                    break
                                case 'tomcat5':
                                    deployScriptPath = "${SERVER_PATH}/tomcat5/deploy.sh"
                                    break
                                case 'tomcat6':
                                    deployScriptPath = "${SERVER_PATH}/tomcat6/deploy.sh"
                                    break
                                case 'tomcat7':
                                    deployScriptPath = "${SERVER_PATH}/tomcat7/deploy.sh"
                                    break
                                case 'tomcat4_local_server':
                                    deployScriptPath = "${SERVER_PATH}/tomcat4_local_server/deploy.sh"
                                    break
                                case 'tomcat-cscac':
                                    deployScriptPath = "${SERVER_PATH}/tomcat-cscac/deploy.sh"
                                    break
                                case 'tomcat-wac':
                                    deployScriptPath = "${SERVER_PATH}/tomcat-wac/deploy.sh"
                                    break
                                case 'tomcat_wat':
                                    deployScriptPath = "${SERVER_PATH}/tomcat_wat/deploy.sh"
                                    break
                                case 'tomcat_wet':
                                    deployScriptPath = "${SERVER_PATH}/tomcat_wet/deploy.sh"
                                    break
                                default:
                                    error "Unknown Tomcat instance: ${params.TOMCAT_INSTANCE}"
                            }

                            // Trigger the deploy.sh script on the selected Tomcat instance using SSH
                            sh """
                                sshpass -p \$SERVER_PASSWORD ssh -o StrictHostKeyChecking=no -p ${SERVER_PORT} ${SERVER_USER}@${SERVER_HOST} 'bash ${deployScriptPath}'
                            """
                        }
                    } else {
                        error "No WAR file found in target directory"
                    }
                }
            }
        }
    }
}

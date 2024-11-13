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
        choice(name: 'TOMCAT_INSTANCE', choices: ['tomcat1', 'tomcat3', 'tomcat4', 'tomcat5', 'tomcat6', 'tomcat7', 'tomcat4_local_server', 'tomcat-cscac', 'tomcat-wac', 'tomcat_wat', 'tomcat_wet', 'tomcat-poc'], 
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
                            // Determine the correct Tomcat folder path based on the selected Tomcat instance
                            def tomcatFolder = ''
                            switch (params.TOMCAT_INSTANCE) {
                                case 'tomcat1':
                                    tomcatFolder = "tomcat1"
                                    break
                                case 'tomcat3':
                                    tomcatFolder = "tomcat3"
                                    break
                                case 'tomcat4':
                                    tomcatFolder = "tomcat4"
                                    break
                                case 'tomcat5':
                                    tomcatFolder = "tomcat5"
                                    break
                                case 'tomcat6':
                                    tomcatFolder = "tomcat6"
                                    break
                                case 'tomcat7':
                                    tomcatFolder = "tomcat7"
                                    break
                                case 'tomcat4_local_server':
                                    tomcatFolder = "tomcat4_local_server"
                                    break
                                case 'tomcat-cscac':
                                    tomcatFolder = "tomcat-cscac"
                                    break
                                case 'tomcat-wac':
                                    tomcatFolder = "tomcat-wac"
                                    break
                                case 'tomcat_wat':
                                    tomcatFolder = "tomcat_wat"
                                    break
                                case 'tomcat_wet':
                                    tomcatFolder = "tomcat_wet"
                                    break
                                case 'tomcat-poc':
                                    tomcatFolder = "tomcat-poc"
                                    break
                                default:
                                    error "Unknown Tomcat instance: ${params.TOMCAT_INSTANCE}"
                            }

                            // Construct the target path to copy the WAR file to the selected Tomcat folder
                            def targetPath = "${SERVER_PATH}/${tomcatFolder}"

                            // Use sshpass to securely copy the WAR file to the appropriate Tomcat folder
                            sh """
                                sshpass -p \$SERVER_PASSWORD scp -o StrictHostKeyChecking=no -rP ${SERVER_PORT} ${warFile} ${SERVER_USER}@${SERVER_HOST}:${targetPath}/
                            """

                            // Trigger the deploy.sh script on the selected Tomcat instance
                            def deployScriptPath = "${SERVER_PATH}/${tomcatFolder}/deploy.sh"
                            sh """
                            # Use sshpass to provide password, and sudo to switch to root
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

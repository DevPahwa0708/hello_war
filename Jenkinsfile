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
        choice(name: 'TOMCAT_INSTANCE', choices: ['tomcat1', 'tomcat2', 'tomcat3', 'tomcat4', 'tomcat5', 'tomcat6', 'tomcat7'], 
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
                        def warFileName = sh(script: "basename ${warFile}", returnStdout: true).trim()

                        echo "WAR file to transfer: ${warFileName}"

                        // Use withCredentials to securely inject the password into the pipeline
                        withCredentials([string(credentialsId: 'pass_wb_uat', variable: 'SERVER_PASSWORD')]) {
                            // Use sshpass to provide the password and copy the WAR file via SCP
                            sh """
                                sshpass -p '\${SERVER_PASSWORD}' scp -rP ${SERVER_PORT} ${warFileName} ${SERVER_USER}@${SERVER_HOST}:
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

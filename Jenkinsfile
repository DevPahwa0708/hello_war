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
        jdk 'java-21'
    }
    environment {
        PROJECT_ID = 'wb-gcp-stg' // Your actual GCP project ID
        REGION = 'asia-south1' // Replace with your desired region
        REPOSITORY = 'wb-staging-repo' // Replace with your Artifact Registry repository name
    }
    stages {
        stage('Build') {
            steps {
                echo "Creating New Build"
                sh 'mvn clean install'
            }
        }
        stage('Authenticate with Google Cloud') {
            steps {
                script {
                    // Authenticate using the service account key
                    withCredentials([file(credentialsId: 'c2021550-56ea-4beb-9c80-35657413c178', variable: 'GOOGLE_CREDENTIALS')]) {
                        sh '''
                        gcloud auth activate-service-account --key-file=$GOOGLE_CREDENTIALS
                        gcloud config set project ${PROJECT_ID}
                        gcloud auth configure-docker ${REGION}-docker.pkg.dev
                        '''
                    }
                }
            }
        }
        stage('Build and Push Docker Image to Artifact Registry') {
            steps {
                script {
                    def customImage = docker.build("${REGION}-docker.pkg.dev/${env.PROJECT_ID}/${env.REPOSITORY}/${env.JOB_NAME.toLowerCase()}:${env.BUILD_NUMBER}")

                    // Push the Docker image to Artifact Registry
                    customImage.push("${env.BUILD_NUMBER}")
                    customImage.push("latest")
                }
            }
        }
        stage('Deploying on UAT') {
            steps {
                sh '/opt/ansible/playbook/deploy.sh'
            }
        }
    }
}

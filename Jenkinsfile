#!/usr/bin/env groovy
import hudson.model.*

pipeline {
    agent any
    tools {
        maven 'maven'
        jdk 'java-21'
    }
    environment {
        AWS_REGION = 'us-east-2' // Replace with your desired AWS region
        ECR_REPOSITORY = 'test' // Replace with your ECR repository name
        AWS_ACCOUNT_ID = '449468931146' // Replace with your AWS account ID
    }
    stages {
        stage('Build') {
            steps {
                echo "Creating New Build"
                sh 'mvn clean install'
            }
        }
        stage('Authenticate with AWS') {
            steps {
                script {
                    // Authenticate using AWS CLI
                    withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-credentials']]) {
                        sh '''
                        aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                        '''
                    }
                }
            }
        }
        stage('Build and Push Docker Image to ECR') {
            steps {
                script {
                    def customImage = docker.build("${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${env.ECR_REPOSITORY}:${env.BUILD_NUMBER}")

                    // Push the Docker image to ECR
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

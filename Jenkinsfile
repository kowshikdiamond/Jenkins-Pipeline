pipeline {
    agent any

    environment {
        AWS_ACCESS_KEY_ID     = credentials('jenkins_aws_access_key_id')
        AWS_SECRET_ACCESS_KEY = credentials('jenkins_aws_secret_access_key')
        ANSIBLE_PRIVATE_KEY   = credentials('aws_private_key')
    }

    stages {
        stage('Install Dependencies') {
            steps {
                script {
                    // Install Python dependencies
                    sh 'pip install boto3 botocore'
                }
            }
        }

        stage('Create Server with Terraform') {
            steps {
                script {
                    // Change to the Terraform directory
                    dir('terraform') {
                        // Initialize Terraform
                        sh 'terraform init'

                        // Apply Terraform configuration
                        sh 'terraform apply --auto-approve'
                    }
                }
            }
        }

        stage('Copy SSH Key to Jenkins Server') {
            steps {
                script {
                    // Change to the Ansible directory
                    dir('ansible') {
                        // Copy SSH key to Jenkins server if it doesn't exist
                        withCredentials([sshUserPrivateKey(credentialsId: 'aws_private_key', keyFileVariable: 'keyfile', usernameVariable: 'user')]) {
                            // Get the current working directory
                            def currentDir = sh(script: 'pwd', returnStdout: true).trim()

                            // Check if the file exists before copying
                            if (!fileExists("${currentDir}/ssh-key.pem")) {
                                // Copy the key to the current working directory
                                sh "cp $KEYFILE ${currentDir}/ssh-key.pem"
                            }
                        }
                    }
                }
            }
        }

        stage('Configure Server with Ansible') {
            steps {
                script {
                    // Change to the Ansible directory
                    dir('ansible') {
                        // Run Ansible playbook
                        sh 'ansible-playbook ansible.yaml'
                    }
                }
            }
        }
    }

    post {
        failure {
            emailext subject: "Jenkins Build Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                      body: "The Jenkins build has failed. Build Status: ${currentBuild.result}",
                      recipientProviders: [
                          [$class: 'CulpritsRecipientProvider'],
                          [$class: 'RequesterRecipientProvider']
                      ]
        }

        success {
            emailext subject: "Jenkins Build Successful: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                      body: "The Jenkins build has completed successfully. Build Status: ${currentBuild.result}",
                      recipientProviders: [
                          [$class: 'CulpritsRecipientProvider'],
                          [$class: 'RequesterRecipientProvider']
                      ]
        }
    }
}

def fileExists(String filePath) {
    return sh(script: "[ -e ${filePath} ]", returnStatus: true) == 0
}
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

        stage('Create and Configure Server') {
            steps {
                script {
                    // Change to the Terraform directory
                    dir('terraform') {
                        // Initialize Terraform
                        sh 'terraform init'

                        // Apply Terraform configuration
                        sh 'terraform apply --auto-approve'
                    }

                    // Change to the Ansible directory
                    dir('ansible') {
                        // Copy SSH key to Jenkins server
                        withCredentials([sshUserPrivateKey(credentialsId: 'aws_private_key', keyFileVariable: 'keyfile', usernameVariable: 'user')]) {
                            // Get the current working directory
                            def currentDir = sh(script: 'pwd', returnStdout: true).trim()

                            // Copy the key to the current working directory
                            sh "cp $KEYFILE ${currentDir}/ssh-key.pem"

                            // Run Ansible playbook
                            sh 'ansible-playbook ansible.yaml'
                        }

                        // Delete the copied SSH key after Ansible playbook
                        sh "rm ${currentDir}/ssh-key.pem"
                    }
                }
            }
        }
    }
}
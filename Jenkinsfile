pipeline {
    agent any

    environment {
        AWS_ACCESS_KEY_ID     = credentials('jenkins_aws_access_key_id')
        AWS_SECRET_ACCESS_KEY = credentials('jenkins_aws_secret_access_key')
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

        stage('Create server') {
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

        stage('Configure server with Ansible') {
            steps {
                script {
                    // Change to the Ansible directory
                    dir('ansible') {
                        // Run Ansible playbook
                        withCredentials([file(credentialsId: 'aws_private_key', variable: 'PRIVATE_KEY_FILE')]) {
                            sh """
                                ansible-playbook -e ANSIBLE_SSH_PRIVATE_KEY="${PRIVATE_KEY_FILE}" ansible.yaml
                            """
                        }
                    }
                }
            }
        }
    }
}
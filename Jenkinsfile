pipeline {
    agent any

    environment {
        AWS_ACCESS_KEY_ID     = credentials('jenkins_aws_access_key_id')
        AWS_SECRET_ACCESS_KEY = credentials('jenkins_aws_secret_access_key')
        ANSIBLE_SSH_PRIVATE_KEY = credentials('aws_private_key')
    }

    stages {
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
                    // Go back to the main project directory
                    dir('..') {
                        // Change to the Ansible directory
                        dir('ansible') {
                            // Run Ansible playbook
                            sh """
                                ansible-playbook -i your_inventory_file \
                                                 -e ANSIBLE_SSH_PRIVATE_KEY="$ANSIBLE_SSH_PRIVATE_KEY" \
                                                 ansible.yaml
                            """
                        }
                    }
                }
            }
        }
    }
}
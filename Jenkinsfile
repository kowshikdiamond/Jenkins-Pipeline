pipeline {
    agent any

    stages {
        stage('provision server') {
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
    }
}

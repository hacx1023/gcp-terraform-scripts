pipeline {
    agent any

    environment {
        # Optional: Terraform binary path if installed manually
        PATH = "/usr/local/bin/:$PATH"
    }

    stages {
        stage('Checkout Code') {
            steps {
                echo "Cloning Terraform project..."
                checkout scm
            }
        }

        stage('Set Azure Credentials') {
            steps {
                withCredentials([
                    string(credentialsId: 'AZURE_SUBSCRIPTION_ID', variable: 'ARM_SUBSCRIPTION_ID'),
                    string(credentialsId: 'AZURE_CLIENT_ID', variable: 'ARM_CLIENT_ID'),
                    string(credentialsId: 'AZURE_CLIENT_SECRET', variable: 'ARM_CLIENT_SECRET'),
                    string(credentialsId: 'AZURE_TENANT_ID', variable: 'ARM_TENANT_ID')
                ]) {
                    sh '''
                        echo "Exporting Azure credentials..."
                        export TF_VAR_subscription_id=$ARM_SUBSCRIPTION_ID
                        export TF_VAR_client_id=$ARM_CLIENT_ID
                        export TF_VAR_client_secret=$ARM_CLIENT_SECRET
                        export TF_VAR_tenant_id=$ARM_TENANT_ID
                        terraform init
                    '''
                }
            }
        }

        stage('Terraform Validate') {
            steps {
                sh 'terraform validate'
            }
        }

        stage('Terraform Plan') {
            steps {
                withCredentials([
                    string(credentialsId: 'AZURE_SUBSCRIPTION_ID', variable: 'ARM_SUBSCRIPTION_ID'),
                    string(credentialsId: 'AZURE_CLIENT_ID', variable: 'ARM_CLIENT_ID'),
                    string(credentialsId: 'AZURE_CLIENT_SECRET', variable: 'ARM_CLIENT_SECRET'),
                    string(credentialsId: 'AZURE_TENANT_ID', variable: 'ARM_TENANT_ID')
                ]) {
                    sh '''
                        export TF_VAR_subscription_id=$ARM_SUBSCRIPTION_ID
                        export TF_VAR_client_id=$ARM_CLIENT_ID
                        export TF_VAR_client_secret=$ARM_CLIENT_SECRET
                        export TF_VAR_tenant_id=$ARM_TENANT_ID
                        terraform plan -out=tfplan
                    '''
                }
            }
        }

        stage('Terraform Apply') {
            steps {
                input message: 'Apply Terraform changes?', ok: 'Apply'
                sh 'terraform apply -auto-approve tfplan'
            }
        }

        stage('Output Public IP') {
            steps {
                echo "Fetching the VM Public IP..."
                sh 'terraform output'
            }
        }
    }

    post {
        success {
            echo "✅ Terraform deployment completed successfully!"
        }
        failure {
            echo "❌ Deployment failed. Please check logs."
        }
    }
}

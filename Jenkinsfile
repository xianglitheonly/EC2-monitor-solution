pipeline {
    agent any
    environment {
        TF_IN_AUTOMATION = 'true'
        TF_CLI_CONFIG_FILE = credentials('tf-creds')
        AWS_SHARED_CREDENTIALS_FILE = credentials('xiangli-admin')
        ANSIBLE_HOST_KEY_CHECKING = 'false'
        
    }
    stages {
        stage('TF Init') {
            steps {
                sh 'ls'
                sh 'cat ./terraform/$BRANCH_NAME.tfvars'
                sh 'cd terraform && terraform init -no-color'
            }
        }
        stage('TF Plan') {
            steps {
                sh 'cd terraform && terraform plan -no-color -var-file="$BRANCH_NAME.tfvars"'
            }
        }
        stage('Checking Plan') {
            when {
                beforeInput true
                branch "dev"
            }
            input {
                message "Is your plan OK?"
                ok "Yes"
            }
            steps {
                echo 'Plan is OK! Starting Ansible deployment!'
            }
        }
        stage('TF Apply') {
            steps {
                sh 'cd terraform && terraform apply -auto-approve -no-color -var-file="$BRANCH_NAME.tfvars"'
            }
        }
        stage('Inventory') {
          steps {
                sh '''cd terraform && printf \\
                    "\\n$(terraform output -json instance_ips | jq -r \'.[]\')" \\
                    >> aws_hosts'''
            }
        }
        stage('EC2 Wait') {
            steps {
                sh '''cd terraform && aws ec2 wait instance-status-ok \\
                      --instance-ids $(terraform output -json instance_ids | jq -r \'.[]\') \\
                      --region us-west-1'''
            }
        }

        stage('Ansible Deploy') {
            steps {
                // sh "echo '\n54.253.71.228' >> ./terraform/aws_hosts"
                ansiblePlaybook(credentialsId: 'ec2-ssh-key', 
                                inventory: './terraform/aws_hosts',
                                playbook: './ansible/main.yaml'
                                )
            }
        }
        stage('Test Grafana and Prometheus') {
          steps {
                ansiblePlaybook(credentialsId: 'ec2-ssh-key',
                                inventory: 'aws_hosts',
                                playbook: './ansible/node-test.yaml'
                                ) 
            }
        }
        stage('Confirm Destroy'){
            when {
                beforeInput true
                branch "dev"
            }
            input {
                message "Are you sure to destroy all?"
                ok "Yes"
            }
            steps {
                echo 'Destroy starting.'
            }
        }
        stage('Destroy') {
            steps {
                sh 'cd terraform && terraform destroy -auto-approve -no-color -var-file="$BRANCH_NAME.tfvars"'
            }
        }
    }
    post {
        success {
            echo 'Success!'
        }
        failure {
            sh 'cd terraform && terraform destroy -auto-approve -no-color -var-file="$BRANCH_NAME.tfvars"'
        }
        aborted {
            sh 'cd terraform && terraform destroy -auto-approve -no-color -var-file="$BRANCH_NAME.tfvars"'
        }
    }
}
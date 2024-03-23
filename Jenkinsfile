pipeline {
    agent any
    environment {
        TF_IN_AUTOMATION = 'true'
        TF_CLI_CONFIG_FILE = credentials('tf-creds')
        AWS_SHARED_CREDENTIALS_FILE = credentials('xiangli-admin')
        
    }
    stages {
        stage('Init') {
            steps {
                sh 'ls'
                sh 'cd terraform && terraform init -no-color'
            }
        }
        stage('Plan') {
            steps {
                sh 'cd terraform && terraform plan -no-color'
            }
        }
        stage('Ansible') {
            steps {
                sh "printf '\n3.27.156.237' >> ./terraform/aws_hosts"
                sh '. /ansible-core-env/bin/activate && \
                    export ANSIBLE_CONFIG=./ansible/ansible.cfg
                    ansible-playbook --key-file /certs/client/id_rsa --user ubuntu ./ansible/main.yaml'
                // ansiblePlaybook(credentialsId: 'ec2-ssh-key', inventory: './terraform/aws_hosts', playbook: './ansible/main.yaml')
            }
        }
        // stage('Validate Apply') {
        //     input {
        //         message "Do you want to Apply this plan?"
        //         ok "Apply"
        //     }
        //     steps {
        //         echo 'Apply Accepted'
        //     }
        // }
        // stage('Apply') {
        //     steps {
        //         sh 'terraform apply -auto-approve -no-color'
        //     }
        // }
        // stage('EC2 Wait') {
        //     steps {
        //          sh 'aws ec2 wait instance-status-ok --region us-west-1'
        //     }
        // }
        // stage('Ansible') {
        //     steps {
        //         ansiblePlaybook(credentialsId: 'mtckey', inventory: 'aws_hosts', playbook: 'playbooks/main-playbook.yml')
        //     }
        // }
        // stage('Destroy') {
        //     steps {
        //         sh 'terraform destroy -auto-approve -no-color'
        //     }
        // }
    }
}
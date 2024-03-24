pipeline {
    agent any
    environment {
        TF_IN_AUTOMATION = 'true'
        TF_CLI_CONFIG_FILE = credentials('tf-creds')
        AWS_SHARED_CREDENTIALS_FILE = credentials('xiangli-admin')
        
    }
    stages {
        // stage('TF Init') {
        //     steps {
        //         sh 'ls'
        //         sh 'cd terraform && terraform init -no-color'
        //     }
        // }
        // stage('TF Plan') {
        //     steps {
        //         sh 'cd terraform && terraform plan -no-color'
        //     }
        // }
        // stage('TF Apply') {
        //     steps {
        //         sh 'cd terraform && terraform apply -auto-approve -no-color'
        //     }
        // }
        // stage('EC2 Wait') {
        //     steps {
        //          sh 'aws ec2 wait instance-status-ok --region ap-southeast-2'
        //     }
        // }
        // stage('Checking EC2') {
        //     input {
        //         message "Are your instances running well?"
        //         ok "Yes"
        //     }
        //     steps {
        //         echo 'EC2 are running well, starting Ansible deployment.'
        //     }
        // }
        stage('Ansible Deploy') {
            steps {
                sh "echo '\n54.253.71.228' >> ./terraform/aws_hosts"
                ansiblePlaybook(credentialsId: 'ec2-ssh-key', 
                                inventory: './terraform/aws_hosts',
                                playbook: './ansible/main.yaml'.
                                extras: ["-e", "ansible_ssh_common_args='-o StrictHostKeyChecking=no'"])
            }
        }
        stage('Checking Applications') {
            input {
                message "Are your applications running well?"
                ok "Yes"
            }
            steps {
                echo 'Applications are running well, starting TF destroy.'
            }
        }
        stage('Destroy') {
            steps {
                sh 'cd terraform && terraform destroy -auto-approve -no-color'
            }
        }

        // stage('Apply') {
        //     steps {
        //         sh 'terraform apply -auto-approve -no-color'
        //     }
        // }

        // stage('Ansible') {
        //     steps {
        //         ansiblePlaybook(credentialsId: 'mtckey', inventory: 'aws_hosts', playbook: 'playbooks/main-playbook.yml')
        //     }
        // }

    }
}
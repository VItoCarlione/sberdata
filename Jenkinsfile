pipeline {
    agent any
    stages{
        stage('SCM Chekout'){
            steps{
                git branch: 'main', url: 'https://github.com/VItoCarlione/sberdata'
            }
        }
        stage('Ansible Exec'){
            steps{
                ansiblePlaybook become: true, credentialsId: 'private-key', disableHostKeyChecking: true, installation: 'ansible3', inventory: 'dev.inv', playbook: 'nginx.yml'
            }
        }
        stage('Test WebServ'){
            steps{
                sh 'curl 87.239.109.136:80'
            }
        }
    }
}

pipeline {
    agent any
    stages {
        stage ('Проверка Hadoop Service') {
            steps {
                parallel (
                    'Проверка доступности Hadoop NameNode': {
                        script{
                            final String url = "http://87.239.109.237:9870/"
                            final def String code =
                                sh(script: "curl -I $url 2>/dev/null | head -n 1 | cut -d ' ' -f2", returnStdout: true).trim()
                                echo "HTTP response status code: $code"
                                if (code == 200){
                                    sh ‘exit 0’
                                    
                                }
                            
                        }
                        
                    },
                    'Проверка доступности Hadoop DataNodes': {
                        script {
                            final String url = "http://87.239.109.237:986/"
                            final def String code =
                                sh(script: "curl -I $url 2>/dev/null | head -n 1 | cut -d ' ' -f2", returnStdout: true).trim()
                                echo "HTTP response status code: $code"
                                if (code == 200){
                                    sh ‘exit 0’
                                    
                                }
                            }
                        }
                    )
                }
            }
            stage('Установка Ngonx с помощью Ansible') {
                steps{
                    ansiblePlaybook become: true, credentialsId: 'private-key', disableHostKeyChecking: true, installation: 'ansible3', inventory: 'dev.inv', playbook: 'nginx.yml'
                    
                }
                
            }
            stage('Проверка доступности Nginx') {
                steps{
                    sh 'curl 87.239.109.136:80'
                    
                }
                
            }
        
    }
    
}

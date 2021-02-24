pipeline {
    agent any
    stages {
        stage ('Проверка Hadoop Service') {
            steps {
                parallel (
                    'Проверка доступности Hadoop NameNode': {
                        script{
                            def String url = "http://87.239.109.237:9870/"
                            def String statusCode = sh(script: "curl --max-time 10 -I $url 2>/dev/null | head -n 1 | cut -d ' ' -f2", returnStdout: true).trim()
                                if (statusCode != "200"){
                                            echo "HTTP response status code: $statusCode"
                                            echo '[FAILURE] Failed to build'
                                            currentBuild.result = 'FAILURE'
                                            throw new Exception("Throw to stop pipeline")
                                }
                            
                        }
                        
                    },
                    'Проверка доступности Hadoop DataNodes': {
                        script {
                            def String url = "http://87.239.109.237:9864/"
                            def String statusCode2 = sh(script: "curl --max-time 10 -I $url 2>/dev/null | head -n 1 | cut -d ' ' -f2", returnStdout: true).trim()
                                if (statusCode2 != "200"){
                                    echo "HTTP response status code: $statusCode2"
                                    echo '[FAILURE] Failed to build'
                                    currentBuild.result = 'FAILURE'
                                    throw new Exception("Throw to stop pipeline")
                                    
                                } 
                            }
                        }
                    )
                }
            }
            stage('Установка Nginx с помощью Ansible') {
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

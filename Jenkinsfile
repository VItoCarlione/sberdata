/*
Pipeline Jenkins для проверки доступности сервисов Hadoop кластера, установки Nginx и проверки его доступности. В целом выполнен в декларативном формате, с элементами шагов 
в скриптовом формате (устаревший формат).
*/

pipeline {
    agent any // Определяем где будет выполняться pipeline, выбрано любой агент, в нашем случае локально на сервере Jenkins
    stages { // Глобальный блок этапов pipeline's
        stage ('Проверка Hadoop Service') { /* Блок 1: проверка в параллели доступности сервисов Hadoop, пока что в виде запроса кода состояния http, 
                                             в процессе доработки проверка целостности hdfs, так же рассматривался вариант запроса состояния сервисов через REST API */
            steps { // Шаг первый с нескольким подэтапами
                parallel ( // Запуск проверок в параллели
                    'Проверка доступности Hadoop NameNode': { // Объявляем название подэтапа
                        script{ // Блок со скриптовой частью, проверяет статус код http, с помощью curl запроса
                            def String url = "http://87.239.109.237:9870/" // Адрес NameNode UI, передается в переменную $url, в виде строкового значения
                            def String statusCode = sh(script: "curl --max-time 10 -I $url 2>/dev/null | head -n 1 | cut -d ' ' -f2", returnStdout: true).trim() /* Запрос данных 
                            через curl c параметрами максимальной длительности коннекта в 10 секунд, если ответа нет этап завершиться с ошибкой, если этого не делать
                            crul будет висеть очень долго в ожидании. head позволяет нам взять первые строки из вывода, с помощью cut образаем нужный статус код*/
                                echo "HTTP response status code: $statusCode"
                                if (statusCode != "200"){
                                            echo '[FAILURE] Failed to build'
                                            currentBuild.result = 'FAILURE'
                                            throw new Exception("Pipeline остановлен виду неуспешного завершения шага!")
                                }
                            
                        }
                        
                    },
                    'Проверка доступности Hadoop DataNodes': { // Объявляем название подэтапа
                        script { // см. выше
                            def String url = "http://87.239.109.237:9864/" // Адрес DataNode UI
                            def String statusCode2 = sh(script: "curl --max-time 10 -I $url 2>/dev/null | head -n 1 | cut -d ' ' -f2", returnStdout: true).trim()
                                echo "HTTP response status code: $statusCode2"
                                if (statusCode2 != "200"){
                                    echo '[FAILURE] Failed to build'
                                    currentBuild.result = 'FAILURE'
                                    throw new Exception("Pipeline остановлен виду неуспешного завершения шага!")
                                    
                                } 
                            }
                        }
                    )
                }
            }
            stage('Установка Nginx с помощью Ansible') {
                steps{
                    ansiblePlaybook become: true, credentialsId: 'private-key', disableHostKeyChecking: true, installation: 'ansible3', inventory: 'dev.inv', playbook: 'nginx.yml'
                    // Запуск playbook хранящегося в репозитории на GitHub, при этом Ansible подключается по SSH ключу
                }
                
            }
            stage('Проверка доступности Nginx') {
                steps{
                    sh 'curl 87.239.109.136:80' // Запрос стартовой странички Nginx, простой вариант проверки без условий
                    
                }
                
            }
        
    }
    
}

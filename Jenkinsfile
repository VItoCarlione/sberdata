/*
Pipeline Jenkins для проверки доступности сервисов Hadoop кластера, установки Nginx и проверки его доступности. В целом выполнен в декларативном формате, с элементами шагов 
в скриптовом формате (устаревший формат).
Установленные плагины: Blue Ocean для визуализации, SSH плагин
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
                            crul будет висеть очень долго в ожидании. head позволяет нам взять первые строки из вывода, с помощью cut образаем нужный статус код
                            returnStdout: true - шаг sh вовзращает null, поэтому ничего не получилось бы без данного параметра, которые позволяет вернуть в переменную полученное значение
                            .trim() - удаляет все новые символы строки, удаляет лишнее */
                                echo "HTTP response status code: $statusCode" // Смотрим какой код вернула страничка
                                if (statusCode != "200"){ // Условный блок через который задаем условие, что если полученный код не равняется 200 (http OK), то выводим инфу о сбое
                                            echo '[FAILURE] Failed to build' // Выводим сообщение о сбое
                                            currentBuild.result = 'FAILURE' // Объяляем что сборка зафэйлилась
                                            throw new Exception("Pipeline остановлен виду неуспешного завершения шага!") // Обрабатываем исключение, через него делаем остановку pipeline
                                }
                            
                        }
                        
                    },
                    'Проверка доступности Hadoop DataNodes': { // Объявляем название подэтапа
                        script { // см. выше
                            def String url = "http://87.239.109.237:9864/" // Адрес DataNode UI
                            def String statusCode2 = sh(script: "curl --max-time 10 -I $url 2>/dev/null | head -n 1 | cut -d ' ' -f2", returnStdout: true).trim() // см. выше пред подэтап
                                echo "HTTP response status code: $statusCode2" // см. выше пред подэтап
                                if (statusCode2 != "200"){ // см. выше пред подэтап
                                    echo '[FAILURE] Failed to build' // см. выше пред подэтап
                                    currentBuild.result = 'FAILURE' // см. выше пред подэтап
                                    throw new Exception("Pipeline остановлен виду неуспешного завершения шага!") // см. выше пред подэтап
                                    
                                } 
                            }
                        } /* В блок parallel в части проверки hdfs добавлен подэтап с проверкой hdfs fsck / через удаленное выполнение команд через SSH плагин         
                    Блок пока закомментирован так как сбоил, нужно еще время чтобы доработать. Альтернативным способом проверки целостности hdfs рассмаривался запрос curl'ом странички
                    с информацией о статус http://87.239.109.237:9870/fsck?ugi=hdoop&path=%2F (%2F = /   urlencode)
                    'Проверка целостности hdfs': { 
                        script {
                            def remote = [:]
                            remote.name = "Hadoop_MCS"
                            remote.host = "87.239.109.237"
                            remote.allowAnyHosts = true
                                withCredentials([sshUserPrivateKey(credentialsId: 'ubuntu', keyFileVariable: 'priv_key', passphraseVariable: '', usernameVariable: 'ubuntu')]) {
                                    remote.user = ubuntu
                                    remote.identityFile = priv_key
                                        sshCommand remote: remote, command: 'hdfs fsck / | tail -n 1 | grep -oE '[^ ]+$'' // Берем вывод команды, читаем последнюю где есть инфа
                                        о состоянии, грепаем последнее слово из строки, так как статус расположен в этой позиции
                        }
                    }                 
                    
                    */
                    )
                }
            }
            stage('Установка Nginx с помощью Ansible') { // Блок 2: Устанавливаем Nginx на удаленный сервер с помощью Ansible
                steps{ // Шаг 1
                    ansiblePlaybook become: true, credentialsId: 'private-key', disableHostKeyChecking: true, installation: 'ansible3', inventory: 'dev.inv', playbook: 'nginx.yml'
                    // Запуск playbook хранящегося в репозитории на GitHub, при этом Ansible подключается по SSH ключу, с повыш привилегиями через become
                }
                
            }
            stage('Проверка доступности Nginx') { /* Блок 3: Проверка доступности HTML странички. Проверить можно аналогичным образом как выше с сервисами Hadoop через 
            http код, либо выполнение удаленной команды systemctl status nginx и если active\running, все хорошо, либо через Ansible Shell Executor или AdHoc команды*/
                steps{
                    sh 'curl 87.239.109.136:80' // Запрос стартовой странички Nginx, простой вариант проверки без условий
                    
                }
                
            }
        
    }
    
}

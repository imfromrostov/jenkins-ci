pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                sh 'docker build -t application .'
            }
        }
        stage('Deploy') {
            steps {
                sh 'docker ps -f name=application -q | xargs --no-run-if-empty docker container stop'
                sh 'docker run --name application --rm -d -p 9889:80 application'
            }
        }
        stage("Test") {
            steps {
                script {
                    final String url = "62.84.118.233:9889"

                    final Integer responseCode = sh(script: "curl -s -o /dev/null -w %{http_code} $url", returnStdout: true).trim()
                    
                    if (responseCode == 200) {
                        echo "Test прошел успешно!"
                    } else {
                        echo responseCode
                        sh 'exit 1'
                    }
                }
            }
        }
        stage("FingerprintTest") {
            steps {
                script {
                    final String url = "62.84.118.233:9889"

                    final String curlFingerprint = sh(script: "wget -q -O- $url | md5sum | cut -c 1-32", returnStdout: true).trim()
                    final String fileFingerprint = sh(script: "md5sum app/index.html | cut -c 1-32", returnStdout: true).trim()
                    
                    if (curlFingerprint == fileFingerprint) {
                        echo "FingerprintTest прошел успешно!"
                    } else {
                        echo "curlFingerprint: $curlFingerprint и fileFingerprint: $fileFingerprint не равны"
                        sh 'exit 1'
                    }
                }
            }
        }
        stage("DeleteContainer") {
            steps {
                sh 'docker ps -f name=application -q | xargs --no-run-if-empty docker container stop'
            }
        }
    }
    
     post {
        success {            
            telegramSend(message: 'Проект собрался успешно!')
        }
        aborted {             
            telegramSend(message: "Отмена сборки проекта!")
        }
        failure {
            telegramSend(message: "Ошибка при сборки проекта!")
        }
    }
}

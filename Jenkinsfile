pipeline {
    agent any
    environment {
    UN_AND_IP = credentials('username_and_ip')
    }
    stages {
        stage ('Building') {
            steps {
                sh './gradlew clean bootJar'
                }
              }
        stage ('Stop old instance') {
            steps {
                sshagent(credentials : ['geodata_key']) {
                    script {
                        try {
                            sh 'ssh -o StrictHostKeyChecking=no ${UN_AND_IP} -T "lsof -i :8080 && fuser -k 8080/tcp"' 
                        }
                        catch (Exception e){
                            echo 'No app running'
                        }
                    }
                    
                }
            }
    }
        stage ('Copy to instance') {
          steps {
                dir ('build/libs') {
                    sshagent(credentials : ['geodata_key']) {
                    sh 'ssh -o StrictHostKeyChecking=no ${UN_AND_IP} -T "rm *.jar || true"'
                    sh 'scp *.jar ${UN_AND_IP}:~/'
                }
            }
          }
        }
        stage ('Run new app') {
          steps {
            sshagent(credentials : ['geodata_key']) {
                sh 'ssh -o StrictHostKeyChecking=no ${UN_AND_IP} -T "nohup java -jar ~/*.jar > rest.log 2>&1&"'
            }
          }
        }
    }
}


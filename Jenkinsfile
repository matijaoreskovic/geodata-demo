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
                    sh 'ssh -o StrictHostKeyChecking=no ${UN_AND_IP} "fuser -f 8080/tcp"' 
                    sh 'ssh -o StrictHostKeyChecking=no ${UN_AND_IP} "if [ $? -eq 0 ]; then fuser -k 8080/tcp; fi"'
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


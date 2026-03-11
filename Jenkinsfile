// Jenkins Global Tool Configuration'da Maven ve JDK 21 tanımlı olmalı (isimler: Maven-3.9, JDK-21)
// İsimler farklıysa tools { } bloğunu kendi kurulumunuza göre düzenleyin veya kaldırıp agent'ta mvn/java path kullanın.
pipeline {
    agent any

    tools {
        maven 'Maven-3.9'
        jdk 'JDK-21'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build & Test') {
            steps {
                sh 'mvn clean package -B -q'
            }
        }
    }

    post {
        success {
            echo 'Build başarılı.'
        }
        failure {
            echo 'Build başarısız.'
        }
    }
}

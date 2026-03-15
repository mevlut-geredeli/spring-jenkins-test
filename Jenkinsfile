pipeline {
    agent any

    environment {
        JAVA_HOME = "/usr/lib/jvm/java-21-openjdk-amd64"
        MAVEN_HOME = "/usr/share/maven"
        PATH = "${JAVA_HOME}/bin:${MAVEN_HOME}/bin:${env.PATH}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean verify -B'
            }
        }

        stage('Static Code Analysis') {
            steps {
                echo "SonarQube analysis hazırlanıyor"
                // sh 'mvn sonar:sonar'
            }
        }

        stage('Package') {
            steps {
                sh 'mvn package -DskipTests'
            }
        }

        stage('Upload Artifact (Nexus)') {
            steps {
                echo "Artifact Nexus'a gönderilecek"
                // sh 'mvn deploy'
            }
        }

        stage('Deploy to Server') {
            steps {
                echo "Remote server deploy"
                // sh 'scp target/app.jar user@server:/opt/app/'
                // sh 'ssh user@server systemctl restart app'
            }
        }

    }

    post {

        success {
            echo 'Pipeline başarıyla tamamlandı.'
        }

        failure {
            echo 'Pipeline başarısız.'
        }

    }
}
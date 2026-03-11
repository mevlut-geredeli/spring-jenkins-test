pipeline {
    agent any

    environment {
        JAVA_HOME = "/usr/lib/jvm/java-21-openjdk-amd64"
        PATH = "${JAVA_HOME}/bin:/usr/share/maven/bin:${env.PATH}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -B -q'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test -B -q'
            }
        }

        stage('Static Code Analysis') {
            steps {
                echo "SonarQube analizi için hazır"
                // sh 'mvn sonar:sonar -Dsonar.projectKey=... -Dsonar.host.url=...'
            }
        }

        stage('Artifact Upload') {
            steps {
                echo "Nexus deploy için hazır"
                // sh 'mvn deploy -DaltDeploymentRepository=...'
            }
        }

        stage('Deploy to Server') {
            steps {
                echo "Uygulama mvlt1 veya hedef server'a deploy edilecek"
                // scp / ansible vb.
            }
        }
    }

    post {
        success { echo 'Pipeline başarıyla tamamlandı.' }
        failure { echo 'Pipeline başarısız.' }
    }
}
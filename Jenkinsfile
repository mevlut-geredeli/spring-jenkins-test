pipeline {
    agent any

    // Deploy ortamını parametre olarak alıyoruz
    parameters {
        choice(name: 'ENV', choices: ['test', 'prod'], description: 'Deploy ortamı seçin')
    }

    environment {
        JAVA_HOME = "/usr/lib/jvm/java-21-openjdk-amd64"
        MAVEN_HOME = "/usr/share/maven"
        PATH = "${JAVA_HOME}/bin:${MAVEN_HOME}/bin:${env.PATH}"

        DEPLOY_HOST = "31.56.60.92"             // Application server IP
        SSH_KEY     = "/var/jenkins_home/deploy_keys/deploy_key"
    }

    options {
        timestamps()
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build & Test') {
            steps {
                sh 'mvn clean verify -B'
            }
        }

        stage('Package') {
            steps {
                sh 'mvn package -DskipTests'
            }
        }

        stage('Prepare Version') {
            steps {
                script {
                    // Versiyon timestamp ile hazırlanıyor
                    env.VERSION = sh(
                        script: "date +%Y%m%d%H%M%S",
                        returnStdout: true
                    ).trim()
                    echo "Build version: ${VERSION}"
                }
            }
        }

        stage('Set Environment Config') {
            steps {
                script {
                    // Test ve prod için path ve service ayrımı
                    if (params.ENV == 'test') {
                        env.APP_PATH     = "/opt/apps/demo-test"
                        env.SERVICE_NAME = "demo-test"
                    } else {
                        env.APP_PATH     = "/opt/apps/demo-prod"
                        env.SERVICE_NAME = "demo-prod"
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                sh """
                    # Klasörleri remote’da oluştur
                    ssh -i ${SSH_KEY} -o StrictHostKeyChecking=no root@${DEPLOY_HOST} 'mkdir -p ${APP_PATH}/releases ${APP_PATH}/logs'

                    # Jar dosyasını remote releases klasörüne kopyala
                    scp -i ${SSH_KEY} -o StrictHostKeyChecking=no target/*.jar root@${DEPLOY_HOST}:${APP_PATH}/releases/app-${VERSION}.jar

                    # Symlink current’i güncelle ve servisi restart et
                    ssh -i ${SSH_KEY} -o StrictHostKeyChecking=no root@${DEPLOY_HOST} '
                        ln -sfn ${APP_PATH}/releases/app-${VERSION}.jar ${APP_PATH}/current
                        systemctl daemon-reload
                        systemctl restart ${SERVICE_NAME}
                    '
                """
            }
        }
    }

    post {
        success {
            echo "Deploy başarılı → ${params.ENV}"
        }
        failure {
            echo "Deploy başarısız → ${params.ENV}"
        }
    }
}
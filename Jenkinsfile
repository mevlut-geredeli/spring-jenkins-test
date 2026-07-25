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

        DEPLOY_HOST = "45.198.68.163"             // Application server IP
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
                    // prod branch'i her zaman prod'a gider; diğer branch'lerde ENV parametresi geçerli.
                    // (Branch taramasının tetiklediği build'ler parametre default'u (test) ile çalıştığından,
                    //  prod branch push'ları eskiden yanlışlıkla test'e deploy oluyordu.)
                    env.DEPLOY_ENV = (env.BRANCH_NAME == 'prod') ? 'prod' : params.ENV

                    // Test ve prod için path, service ve port ayrımı
                    // (iki servis aynı sunucuda çalıştığı için prod 8081'e alındı)
                    if (env.DEPLOY_ENV == 'test') {
                        env.APP_PATH     = "/opt/apps/demo-test"
                        env.SERVICE_NAME = "demo-test"
                        env.APP_PORT     = "8080"
                    } else {
                        env.APP_PATH     = "/opt/apps/demo-prod"
                        env.SERVICE_NAME = "demo-prod"
                        env.APP_PORT     = "8081"
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

                    # Symlink current’i güncelle, portu systemd override ile sabitle ve servisi restart et
                    ssh -i ${SSH_KEY} -o StrictHostKeyChecking=no root@${DEPLOY_HOST} '
                        ln -sfn ${APP_PATH}/releases/app-${VERSION}.jar ${APP_PATH}/current
                        mkdir -p /etc/systemd/system/${SERVICE_NAME}.service.d
                        {
                            echo "[Service]"
                            echo "ExecStart="
                            echo "ExecStart=/usr/bin/java -jar ${APP_PATH}/current --server.port=${APP_PORT}"
                        } > /etc/systemd/system/${SERVICE_NAME}.service.d/override.conf
                        systemctl daemon-reload
                        systemctl restart ${SERVICE_NAME}
                    '
                """
            }
        }

        stage('Health Check') {
            steps {
                sh """
                    ssh -i ${SSH_KEY} -o StrictHostKeyChecking=no root@${DEPLOY_HOST} '
                        for i in 1 2 3 4 5 6 7 8 9 10; do
                            sleep 3
                            if curl -sf http://localhost:${APP_PORT}/ > /dev/null; then
                                echo "Health check OK (port ${APP_PORT})"
                                exit 0
                            fi
                        done
                        echo "Health check FAILED (port ${APP_PORT})"
                        systemctl status ${SERVICE_NAME} --no-pager -n 10
                        exit 1
                    '
                """
            }
        }
    }

    post {
        success {
            echo "Deploy başarılı → ${env.DEPLOY_ENV}"
        }
        failure {
            echo "Deploy başarısız → ${env.DEPLOY_ENV}"
        }
    }
}
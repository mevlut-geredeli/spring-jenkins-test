// Dummy change - pipeline config
pipeline {

    agent any

    environment {

        JAVA_HOME = "/usr/lib/jvm/java-21-openjdk-amd64"
        MAVEN_HOME = "/usr/share/maven"
        PATH = "${JAVA_HOME}/bin:${MAVEN_HOME}/bin:${env.PATH}"

        DEPLOY_HOST = "31.56.60.92"
        APP_PATH = "/opt/apps/demo"

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

        stage('Build & Unit Test') {
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
                    env.VERSION = sh(
                        script: "date +%Y%m%d%H%M%S",
                        returnStdout: true
                    ).trim()
                }
            }
        }

        stage('Deploy TEST') {

            when {
                branch 'test'
            }

            steps {

                echo "Deploying version ${VERSION} to TEST"

                sshagent(['deploy-key']) {

                    sh """
                    scp target/*.jar root@${DEPLOY_HOST}:${APP_PATH}/releases/app-${VERSION}.jar
                    """

                    sh """
                    ssh root@${DEPLOY_HOST} '

                        ln -sfn ${APP_PATH}/releases/app-${VERSION}.jar ${APP_PATH}/current
                        systemctl restart demo

                    '
                    """

                }

            }

        }

        stage('Deploy PROD') {

            when {
                branch 'prod'
            }

            steps {

                echo "Deploying version ${VERSION} to PROD"

                sshagent(['deploy-key']) {

                    sh """
                    scp target/*.jar root@${DEPLOY_HOST}:${APP_PATH}/releases/app-${VERSION}.jar
                    """

                    sh """
                    ssh root@${DEPLOY_HOST} '

                        ln -sfn ${APP_PATH}/releases/app-${VERSION}.jar ${APP_PATH}/current
                        systemctl restart demo

                    '
                    """

                }

            }

        }

    }

    post {

        success {

            echo "Pipeline başarılı"

        }

        failure {

            echo "Pipeline başarısız"

        }

    }

}
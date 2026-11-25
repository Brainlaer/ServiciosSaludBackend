pipeline {
    agent any

    triggers {
        githubPush()
        pollSCM('H/5 * * * *')
    }

    options {
        disableConcurrentBuilds()
        timestamps()
    }

    environment {
        FRONTEND_REPOSITORY = 'https://github.com/Brainlaer/ServiciosSaludFrontend.git'
        BACKEND_IMAGE = 'serviciosalud-backend'
        FRONTEND_IMAGE = 'serviciosalud-frontend'
        IMAGE_TAG = "${BUILD_NUMBER}"
        COMPOSE_PROJECT_NAME = "serviciosalud-ci-${BUILD_NUMBER}"
        POSTGRES_DB = 'salud_ci'
        POSTGRES_USER = credentials('ci-postgres-user')
        POSTGRES_PASSWORD = credentials('ci-postgres-password')
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                dir('frontend') {
                    git branch: 'main',
                        credentialsId: 'github-https',
                        poll: true,
                        url: env.FRONTEND_REPOSITORY
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    docker.build("${env.BACKEND_IMAGE}:${env.IMAGE_TAG}", '.')
                    docker.build("${env.FRONTEND_IMAGE}:${env.IMAGE_TAG}", 'frontend')
                }
            }
        }

        stage('Test') {
            steps {
                sh '''
                    set -eu
                    docker compose -p "$COMPOSE_PROJECT_NAME" -f docker-compose.ci.yml up -d --no-build

                    attempt=1
                    while [ "$attempt" -le 30 ]; do
                        if docker compose -p "$COMPOSE_PROJECT_NAME" -f docker-compose.ci.yml exec -T frontend \
                            wget -q -O /dev/null http://backend:8082/api/servicio/listaServicioDto2; then
                            echo "Frontend container reached backend by its Compose service name."
                            docker compose -p "$COMPOSE_PROJECT_NAME" -f docker-compose.ci.yml exec -T frontend \
                                wget -q -O /dev/null http://localhost/
                            docker compose -p "$COMPOSE_PROJECT_NAME" -f docker-compose.ci.yml exec -T frontend \
                                wget -q -O /dev/null http://localhost/api/servicio/listaServicioDto2
                            echo "Frontend page and reverse proxy are responding."
                            exit 0
                        fi
                        echo "Waiting for backend and database (attempt $attempt/30)..."
                        sleep 2
                        attempt=$((attempt + 1))
                    done

                    echo "Frontend could not reach the backend API after 60 seconds."
                    exit 1
                '''
            }
        }
    }

    post {
        always {
            script {
                def compose = "docker compose -p ${env.COMPOSE_PROJECT_NAME} -f docker-compose.ci.yml"
                def logsStatus = sh(script: "${compose} logs --no-color", returnStatus: true)
                if (logsStatus != 0) {
                    echo "Could not collect Compose logs (exit ${logsStatus})."
                }

                def cleanupStatus = sh(
                    script: "${compose} down --volumes --remove-orphans",
                    returnStatus: true
                )
                if (cleanupStatus != 0) {
                    echo "Compose cleanup did not complete successfully (exit ${cleanupStatus})."
                }
            }
        }
        success {
            echo 'Both images built and the service communication checks passed.'
        }
        failure {
            echo 'CI failed. Review the stage output and Compose logs above.'
        }
    }
}

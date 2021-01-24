pipeline {
    agent any
    stages {
        stage("Build") {
            environment {
                DB_HOST = "127.0.0.1"
                DB_DATABASE = "laravel"
                DB_USERNAME = "root"
                DB_PASSWORD = ""
            }
            steps {
                sh 'php --version'
                sh 'composer install'
                sh 'composer --version'
                sh 'cp .env.example .env'
                sh 'echo DB_HOST=${DB_HOST} >> .env'
                sh 'echo DB_USERNAME=${DB_USERNAME} >> .env'
                sh 'echo DB_DATABASE=${DB_DATABASE} >> .env'
                sh 'echo DB_PASSWORD=${DB_PASSWORD} >> .env'
                sh 'php artisan key:generate'
                sh 'cp .env .env.testing'
            }
        }
        stage("Unit test") {
            steps {
                sh 'php artisan test'
            }
        }
        stage("Code coverage") {
            steps {
                sh "vendor/bin/phpunit --coverage-html 'reports/coverage'"
            }
        }
        stage("Static code analysis larastan") {
            steps {
                sh "vendor/bin/phpstan analyse --memory-limit=2G"
            }
        }
        stage("Static code analysis phpcs") {
            steps {
                sh "vendor/bin/phpcs"
            }
        }
        stage("Docker build") {
            steps {
                sh "docker build -t ahmedassimi/laravelApp ."
            }
        }
        stage("Docker push") {
            environment {
                DOCKER_USERNAME = "ahmedassimi"
                DOCKER_PASSWORD = "123456789a-e"
            }
            steps {
                sh "docker login --username ${DOCKER_USERNAME} --password ${DOCKER_PASSWORD}"
                sh "docker push ahmedassimi/laravelApp"
            }
        }
        stage("Deploy to staging") {
            steps {
                sh "docker run -d --rm -p 8081:80 --name laravelApp ahmedassimi/laravelApp"
            }
        }
        stage("Acceptance test curl") {
            steps {
                sleep 20
                sh "chmod +x acceptance_test.sh && ./acceptance_test.sh"
            }
        }
        stage("Acceptance test codeception") {
            steps {
                sh "vendor/bin/codecept run"
            }
            post {
                always {
                    sh "docker stop laravelApp"
                }
            }
        }
    }
}
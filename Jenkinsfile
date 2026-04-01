/**
 * CI/CD Pipeline - Microservices Lab
 *
 * This pipeline builds and tests three microservices (Go, PHP, Python),
 * each running inside its own ephemeral Docker container.
 *
 * Architecture:
 *   - go-app/      -> golang:1.21-alpine
 *   - php-app/     -> php:8.2-cli-alpine
 *   - python-app/  -> python:3.12-alpine
 *
 * Each stage reuses the workspace cloned in the first stage (reuseNode: true)
 * so the source code is available without cloning again.
 */

pipeline {
    // No default agent — each stage runs in its own Docker container.
    agent none

    options {
        timeout(time: 1, unit: 'HOURS')
        disableConcurrentBuilds()
    }

    stages {

        stage('Checkout') {
            agent any
            steps {
                checkout scm
            }
        }

        stage('Go - Test & Build') {
            agent {
                docker {
                    image 'golang:1.21-alpine'
                    reuseNode true
                }
            }
            environment {
                GOCACHE = "${WORKSPACE}/.cache/go-build"
                GOPATH = "${WORKSPACE}/.cache/go"
            }
            steps {
                echo '=== Go: Test & Build ==='
                sh 'go version'
                dir('go-app') {
                    sh 'go build -o app main.go'
                    sh './app'
                }
            }
        }

        stage('PHP - Test') {
            agent {
                docker {
                    image 'php:8.2-cli-alpine'
                    reuseNode true
                }
            }
            steps {
                echo '=== PHP: Test ==='
                sh 'php -v'
                dir('php-app') {
                    sh 'php index.php'
                }
            }
        }

        stage('Python - Test') {
            agent {
                docker {
                    image 'python:3.12-alpine'
                    reuseNode true
                }
            }
            steps {
                echo '=== Python: Test ==='
                sh 'python --version'
                dir('python-app') {
                    sh 'python main.py'
                }
            }
        }
    }

    post {
        always {
            node('') {
                echo 'Cleaning up workspace...'
                cleanWs()
            }
        }
        success {
            githubNotify context: 'Pruebas y Build', status: 'SUCCESS', description: 'Todo verde'
        }
        failure {
            githubNotify context: 'Pruebas y Build', status: 'FAILURE', description: 'Fallaron los tests'
        }
    }
}

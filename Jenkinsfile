pipeline {
    agent any

    environment {
        GITHUB_TOKEN = credentials('github-token')
    }

    stages {
        stage('Set pending') {
            steps {
                githubNotify context: 'jenkins-pr-validation', status: 'PENDING'
            }
        }

        stage('Build & Test') {
            steps {
                sh './run-tests.sh'
            }
        }

        stage('Set success') {
            steps {
                githubNotify context: 'jenkins-pr-validation', status: 'SUCCESS'
            }
        }
    }

    post {
        failure {
            githubNotify context: 'jenkins-pr-validation', status: 'FAILURE'
        }
    }
}

pipeline {
  agent any
    tools {
        nodejs 'node 18.5.0'
    }

    stages {
        stage('Clean folder') {
            steps {
                 cleanWs()
            }
        }
        stage('Get last commit ID') {
            steps {
                checkout scm
                sh 'git rev-parse --short HEAD > .git/commit-id'
                script {
                    commit_id = readFile('.git/commit-id').trim()
                }
            }
        }
        stage('npm install') {
            steps {
                sh 'npm install'
            }
        }
        stage('npm build') {
            steps {
                sh 'npm run build'
            }
        }
        stage('Docker image build') {
            steps {
                sh "docker build -t fleetman-webapp:${commit_id} ."
            }
        }
    }
}

pipeline {
    agent any
    tools {
        nodejs 'node 18.5.0'
    }
    stages {
        stage('Get last commit ID') {
            steps {
                checkout scm
                sh 'git rev-parse --short HEAD > .git/commit-id'
                script {
                    commit_id = readFile('.git/commit-id').trim()
                }
            }
        }
        stages('npm install') {
            steps {
                sh 'npm install'
            }
        }
        stages('npm build') {
            steps {
                sh 'npm run build'
            }
        }
}

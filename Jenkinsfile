pipeline {
  agent any
    tools {
        nodejs 'node 16.16.0'
    }

    stages {
        stage('Clean folder') {
            steps {
                 cleanWs()
            }
        }
        stage('SonarQube Scan Code Quality') {
            agent any
            steps {
                withSonarQubeEnv(installationName:'sonarqubeIns')
                sh 'npm install sonar-scanner'
                sh 'npm run sonar'
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
                sh 'npm run build --prod'
            }
        }
        stage('Docker image build') {
            steps {
                sh "docker build -t fleetman-webapp:${commit_id} ."
            }
        }
    }
}

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
            def scannerHome = tool 'SonarScanner 4.0';
            steps {
                withSonarQubeEnv('sonarqubeIns') {
                  sh "${scannerHome}/bin/sonar-scanner"
                }
            }
        }
        stage("Quality Gate") {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
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

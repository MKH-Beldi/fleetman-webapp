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
        stage('Get last commit ID') {
            steps {
                checkout scm
                sh 'git rev-parse --short HEAD > .git/commit-id'
                script {
                    commit_id = readFile('.git/commit-id').trim()
                }
            }
        }
        stage('SonarQube Scan Code Quality') {
            steps {
                script {
                    scannerHome = tool 'SonarScanner 4.0';
                }
                withSonarQubeEnv('sonarqubeIns') {
                  sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=fleetman-webapp-test -Dsonar.language=ts -Dsonar.analysis.mode=publish  -Dsonar.sources=src -Dsonar.typescript.lcov.reportPaths=coverage/lcov.info -Dsonar.inclusions=**/*.spec.ts -Dsonar.exclusions=node_modules/*,**/*.spec.ts"
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

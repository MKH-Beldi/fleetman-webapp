pipeline {
  agent any
    tools {
        nodejs 'node 16.16.0'
    }
    environment {
        imageName = "fleetman-webapp"
        registryCredentials = "nexus"
        registry = "nexus-registry.eastus.cloudapp.azure.com:8085/"
        dockerImage = ''
    }

    stages {
        stage('Clean Workspace') {
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
                  sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=fleetman-webapp -Dsonar.language=ts -Dsonar.analysis.mode=publish -Dsonar.sources=src -Dsonar.typescript.lcov.reportPaths=coverage/lcov.info -Dsonar.exclusions=node_modules/*,**/*.spec.ts"
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
                 script {
                    dockerImage = docker.build imageName + ":${commit_id}"
                 }
            }
        }
        stage('Push Docker image to Nexus Registry') {
            steps {
                script {
                    docker.withRegistry( 'http://'+registry, registryCredentials,  )
                    dockerImage.push()
                }
            }
        }

    }
}

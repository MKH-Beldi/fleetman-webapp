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
        stage('Git Preparation') {
            steps {
                cleanWs()
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
        stage("Quality Gate from SonarQube") {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage('Install dependencies') {
            steps {
                sh 'npm install'
            }
        }
        stage('Test Karma') {
          steps {
              sh 'npm test'
          }
        }
        stage('Build') {
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
                    docker.withRegistry( 'http://'+registry, registryCredentials) {
                        dockerImage.push()
                        dockerImage.push("latest")
                    }
                }
            }
        }
        stage('Trigger K8S Manifest Update') {
            steps {
                build job: 'k8s-update-manifests-fleetman-webapp', parameters: [string(name: 'DOCKERTAG', value: commit_id)]
            }

        }
    }
}

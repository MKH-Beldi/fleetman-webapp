pipeline {
  agent any
    tools {
        nodejs 'node 16.16.0'
    }
    environment {
        imageName = "fleetman-webapp"
        registryCredentials = "nexus"
        registry = ''
        dockerImage = ''
    }
    stages {

        stage("Set environment Develop") {
             when {
                branch "feature/*"
             }
            steps{
                 script {
                    registry = "nexus-registry.eastus.cloudapp.azure.com:8086/"
                 }
            }
        }

        stage("Set environment QA") {
             when {
                branch "release/*"
             }
            steps{
                 script {
                    registry = "nexus-registry.eastus.cloudapp.azure.com:8088/"
                 }
            }
        }

        stage('Git Preparation') {
            steps {
                cleanWs()
                checkout scm
                sh 'git rev-parse --short HEAD > .git/commit-id'
                script {
                    commit_id = readFile('.git/commit-id').trim()
                    branch_git = env.BRANCH_NAME
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

        stage('Build') {
            steps {
                sh 'npm run build --prod'
            }
        }

        stage('Build Docker image environment Develop') {
            when {
                branch "feature/*"
            }
            steps {
                script {
                    dockerImage = docker.build imageName + ":${commit_id}-dev"
                }
            }
        }

        stage('Build Docker image environment QA') {
            when {
                branch "release/*"
            }
            steps {
                script {
                    dockerImage = docker.build imageName + ":${commit_id}-test"
                }
            }
        }

        stage('Push Docker image to Nexus Registry') {
            steps {
                script {
                    docker.withRegistry( 'https://'+registry, registryCredentials) {
                        dockerImage.push()
                        dockerImage.push("latest")
                    }
                }
            }
        }

        stage('Trigger K8S Manifest Updating') {
            steps {
                build job: 'k8s-update-manifests-fleetman-webapp', parameters: [string(name: 'DOCKERTAG', value: commit_id), string(name: 'BRANCH_GIT', value: branch_git)]
            }
        }
    }
}

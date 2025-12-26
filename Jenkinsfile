pipeline {
    agent any
    tools {
        jdk 'jdk-21'
        nodejs 'node'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        // Adding the PORT env variable we missed earlier
        PORT = "3000" 
    }
    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/waseem00096/netflix-clone.git'
            }
        }
        stage("Sonarqube Analysis") {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=netflix-clone \
                    -Dsonar.projectKey=netflix-clone'''
                }
            }
        }
        stage("Quality Gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token' 
                }
            } 
        }
        stage('Install Dependencies') {
            steps {
                // Added legacy-peer-deps to avoid the ERESOLVE error
                sh "npm install --legacy-peer-deps"
            }
        }        
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        stage("Docker Build & Push") {
            steps {
                script {
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {   
                       sh "docker build -t waseem09/netflix-clone:latest ."
                       sh "docker push waseem09/netflix-clone:latest"
                    }
                }
            }
        }
       
        stage("TRIVY Image Scan") {
            steps {
                sh "trivy image waseem09/netflix-clone:latest > trivyimage.txt" 
            }
        }
        stage('Cleanup & Deploy') {
            steps {
                script {
                    // Remove old container if it exists to avoid port/name conflicts
                    sh "docker rm -f netflix-clone-app || true"
                    // Pass the PORT environment variable to fix the "argument missing" error
                    sh 'docker run -d --name netflix-clone-app -p 8081:80 -e PORT=80 waseem09/netflix-clone:latest'
                }
            }
        }
    }
    post {
        always {
            // Important: Clean up dangling images to save your 3.3Gi RAM
            sh 'docker image prune -f'
        }
    }
}

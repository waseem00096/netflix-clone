pipeline {
    agent any
    tools {
        jdk 'jdk-21'
        nodejs 'node'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
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
                       // API key is correctly injected here
                       sh "docker build --build-arg TMDB_V3_API_KEY=0241f339597a981eef7440309193c7c5 -t waseem09/netflix-clone:latest ."
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
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh '''
                    export KUBECONFIG=/var/lib/jenkins/.kube/config
                    # Ensure the case matches your folder (Kubernetes vs kubernetes)
                    kubectl apply -f Kubernetes/manifest.yml
                    kubectl rollout status deployment/netflix-deployment
                    '''
                }
            }
        }
    } // End of Stages
    post {
        always {
            // Reclaims space on your 3.3Gi RAM server
            sh 'docker image prune -f'
        }
    }
}

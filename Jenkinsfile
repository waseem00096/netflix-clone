pipeline{
    agent any
    tools{
        jdk 'jdk-21'
        nodejs 'node'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
        stage('clean workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout from Git'){
            steps{
                git branch: 'main', url: 'https://github.com/waseem00096/netflix-clone.git'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('SonarQube') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=netflix-clone  \
                    -Dsonar.projectKey=netflix-clone'''
                }
            }
        }
        stage("quality gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token' 
                }
            } 
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }        
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        stage("Docker Build & Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){   
                       sh "docker build -t netflix ."
                       sh "docker tag netflix waseem09/netflix-clone:latest "
                       sh "docker push waseem09/netflix-clone:latest "
                    }
                }
            }
        }
		stage('Docker Scout Image') {
            steps {
                script{
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){
                       sh 'docker-scout quickview waseem09/netflix-clone:latest'
                       sh 'docker-scout cves waseem09/netflix-clone:latest'
                       sh 'docker-scout recommendations waseem09/netflix-clone:latest'
                   }
                }
            }
        }

        stage("TRIVY-docker-images"){
            steps{
                sh "trivy image waseem09/netflix-clone:latest > trivyimage.txt" 
            }
        }
        stage('App Deploy to Docker container'){
            steps{
                sh 'docker run -d --name netflix-clone-app -p 3000:3000 waseem09/netflix-clone:latest'
            }
        }

    }
   

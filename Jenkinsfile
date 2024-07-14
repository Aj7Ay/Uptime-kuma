pipeline{
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'node18'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
        stage ("Git Pull"){
            steps{
                git branch: 'main', url: 'https://github.com/Olalere1/Uptime-kuma.git'
            }
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Chatbot \
                    -Dsonar.projectKey=Chatbot '''
                }
            }
        }
        stage('Sonar-quality-gate') {
            steps {
                script{
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token'
                }
            }
        }
        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.json"
            }
        }
        stage("Docker Build & Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){   
                       sh "docker build -t uptime ."
                       sh "docker tag uptime olalere1/uptime:latest "
                       sh "docker push olalere1/uptime:latest "
                    }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image olalere1/uptime:latest > trivy.json" 
            }
        }
        stage ("Remove container") {
            steps{
                sh "docker stop uptime | true"
                sh "docker rm uptime | true"
             }
        }
        stage('Deploy to container'){
            steps{
                sh 'docker run -d --name uptime -v /var/run/docker.sock:/var/run/docker.sock -p 3001:3001 olalere1/uptime:latest'
            }
        }
    }
}
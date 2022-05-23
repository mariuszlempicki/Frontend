def imageName="192.168.44.44:8082/docker_registry/frontend"
def dockerRegistry="https://192.168.44.44:8082"
def registryCredentials="artifactory"
def dockerTag=""


pipeline {
    agent {
        label "agent"
    }
    
    environment {
        scannerHome = tool 'SonarQube'
    }

    stages {
        stage('Hello') {
            steps {
                echo 'Start Frontend Pipe'
            }
            
        }
        stage('Clone Frontend repo') {
            steps {
                // git branch: 'main', url: 'https://github.com/mariuszlempicki/Frontend.git'
                checkout scm
            }
        }
        
        stage('Files') {
            steps {
                sh "ls -la"
            }
        }  
        
        stage('Run tests') {
            steps {
                sh "pip3 install -r requirements.txt"
                sh "python3 -m pytest --cov=. --cov-report xml:test-results/coverage.xml --junitxml=test-results/pytestreport.xml"
            }
        }   
        
        stage('SonarQube analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh "${scannerHome}/bin/sonar-scanner"
                }
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage('Build application image') {
            steps {
                script {
                  // Prepare basic image for application
                  dockerTag = "RC-${env.BUILD_ID}"
                  applicationImage = docker.build("$imageName:$dockerTag",".")
                }
            }
        }
        
        stage('Pushing image to Artifactory') {
            steps {
                script {
                  docker.withRegistry("$dockerRegistry", "$registryCredentials"){
                      applicationImage.push();
                      applicationImage.push("latest");
                  }
                }
            }
        }
    }
    
    post {
        always {
            junit testResults: "test-results/*.xml"
            cleanWs()
        }
    }
}
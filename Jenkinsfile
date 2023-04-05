def imageName="lukaszpanda/backend"
def dockerRegistry=""
def registryCredentials="dockerhub"
def dockerTag=""



pipeline {
    agent {
        label 'agent'
    }
    
    
    post {
        always {
            junit testResults: "test-results/*.xml"
            cleanWs()
        }
    }

    
    environment {
        scannerHome = tool 'SonarQube'
    }

    stages {
        stage('Hello') {
            steps {
                echo 'Hello World'
            }
        }
        
        stage('Git Pull backend') {
            steps {
                //git branch: 'tests', url: 'https://github.com/pandalukasz/backend.git'
                checkout scm // Get some code from a GitHub repository
            }
        }
        
        stage('Test app') {
            steps {
                sh 'pip3 install -r requirements.txt' 
                sh 'python3 -m pytest --cov=. --cov-report xml:testresults/coverage.xml --junitxml=test-results/pytestreport.xml'
            }
        }
        
        stage("build & SonarQube analysis") {
            steps {
              withSonarQubeEnv('SonarQube') {
                sh "${scannerHome}/bin/sonar-scanner"
              }
            }
          }
          stage("Quality Gate") {
            steps {
              timeout(time: 1, unit: 'MINUTES') {
                waitForQualityGate abortPipeline: true
              }
            }
          }
          stage('Build application image') {
            steps {
                script {
                    dockerTag = "RC-${env.BUILD_ID}" 
                    applicationImage=docker.build("$imageName:$dockerTag")
                }
            }
          }
          
          stage('Artifactory upload images') {
            steps {
                script {
                    docker.withRegistry("$dockerRegistry", "$registryCredentials"){
                    applicationImage.push()
                    applicationImage.push("latest")
                    }
                }
            }
          }

    }
    
    
}

pipeline{
    
    agent any 
    tools { 
      maven 'maven' 
      jdk 'java' 
    }
    
    stages {
        
        stage('Git Checkout'){
            
            steps{
                
                script{
                    
                    git branch: 'main', url: 'https://github.com/bhattur/demo-counter-app.git'
                }
            }
        }
        stage('UNIT testing'){
            
            steps{
                
                script{
                    
                    sh 'mvn test'
                }
            }
        }
        stage('Integration testing'){
            
            steps{
                
                script{
                    
                    sh 'mvn verify -DskipUnitTests'
                }
            }
        }
        stage('Maven build'){
            
            steps{
                
                script{
                    
                    sh 'mvn clean install'
                }
            }
        }
        stage('Static code analysis'){
            
            steps{
                
                script{
                    
                    withSonarQubeEnv(credentialsId: 'sonartoken') {
                        
                        sh 'mvn clean package sonar:sonar'
                    }
                   }
                    
                }
            }
        stage('Quality Gate Status'){
                
            steps{
                    
                script{
                        
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonartoken'
                }
            }
        }
        stage("upload war file to nexus"){
            steps{
                script{
                    def readPomVersion = readMavenPom file: 'pom.xml'
                    nexusArtifactUploader artifacts: [[artifactId: 'springboot', classifier: '', file: 'target/Uber.jar', type: 'jar']], credentialsId: 'nexus', groupId: 'com.example', nexusUrl: '192.168.29.91:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'demoapp-release', version: "${readPomVersion.version}"
                }
            }
        }
        stage("Docker image build "){
            steps{
                script{
                    sh 'docker image build -t $JOB_NAME:v1.$BUILD_ID .'
                    sh 'docker image tag $JOB_NAME:v1.$BUILD_ID bhatturavishankar/$JOB_NAME:latest'
                    sh 'docker image tag $JOB_NAME:v1.$BUILD_ID bhatturavishankar/$JOB_NAME:v1.$BUILD_ID'
                }
            }
        }
        stage("push image to docker hub"){
            steps{
                script{
                    withCredentials([string(credentialsId: 'docker_creds', variable: 'docker_hub_cred')]) {
                        sh 'docker login -u bhatturavishankar -p ${docker_hub_cred}'
                        sh 'docker image push bhatturavishankar/$JOB_NAME:latest'
                        sh 'docker image push bhatturavishankar/$JOB_NAME:v1.$BUILD_ID'

                }
                }
            }
        }
    } 
        
}

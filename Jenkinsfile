pipeline {
    agent any
    tools{
        jdk "jdk17"
        maven "maven3"
    }
      environment {
          SCANNER_HOME= tool 'sonar-scanner'
      }
   stages {
        stage('git checkout') {
            steps {
                git 'https://github.com/rajdeepsingh642/gaming-board.git'
            }
        }
        stage('compile') {
            steps {
             sh "mvn compile"
             }
        }    
       
        stage('test') {
            steps {
             sh "mvn test"
            }
        }


         
        stage('File system scan') {
            steps {
             sh "trivy fs --format table -o trivy-fs-report.html ."
            }
        }

        
        stage('sonarqube analysis') {
            steps {
             withSonarQubeEnv('sonar') {
                 sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=gaming-demo -Dsonar.projectKey=gaming-demo \
                        -Dsonar.java.binaries=.'''
                    } 
            }
        }
         
         stage('Quality Gate') {
            steps {
               waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
            }
        }
         stage('build stage') {
            steps {
             sh "mvn package"
            }
        }
         stage('publish artifact to nexus') {
            steps {
               withMaven(globalMavenSettingsConfig: 'globalset', jdk: 'jkd17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                sh "mvn deploy"
                   } 
             
            }
        }
           stage('build docker image and tag') {
            steps {
                script{
                   withDockerRegistry(credentialsId: 'docker-hub') {
                    sh "docker build -t rajdeepsingh642/gaming-app:latest ."
                    
                       }
                }
             
            }
        }
            stage('Docker image scan') {
            steps {
             sh "trivy image  --format table -o trivy-fs-report.html  rajdeepsingh642/gaming-app:latest ."
            }
        }
             stage('Docker image push') {
            steps {
                script{
                   withDockerRegistry(credentialsId: 'docker-hub') {
                    sh "docker push rajdeepsingh642/gaming-app:latest"
                    
                       }
                }
             
            }
        }
         
          
            stage('Deploy to k8s') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: ' kubernetes', contextName: '', credentialsId: 'k8s-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://192.168.177.132:6443') {
                sh "kubectl apply -f deployment-service.yaml"
               }
           
            }
        }
            stage('veryfy the deployment') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: ' kubernetes', contextName: '', credentialsId: 'k8s-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://192.168.177.132:6443') {
                sh "kubectl get pod -n webapps"
                sh "kubectl get  svc -n webapps"
            
               }
           
            }
        }
    
     } 
  
       
   }


pipeline {
    agent any
    
    tools{
        maven 'maven3'
        
    }
    
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        
    }

    stages {
        stage('Git Clone') {
            steps {
                git branch: 'main', url: 'https://github.com/mohitekv/Multi-Tier-With-Database.git'
            }
        }
        
        stage('Compile') {
            steps {
                sh "mvn compile"    
            }
        }
        
        stage('Tests') {
            steps {
                sh "mvn test"
            }
        }
        
        stage('Trivy FS scan') {
            steps {
                sh "trivy fs --format table -o fs.html ."
            }
        }
        
        
        stage('SonnarQube') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=bankapp -Dsonar.projectName=bankapp \
                        -Dsonar.java.binaries=target'''
            }
            }
        }
        
        
        stage('Build & Publish to Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'devopsshack-settings', jdk: '', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                sh "mvn deploy"
            }
            }
        }
        
        stage('Build & Tag Docker Image') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cert') {
                        sh "docker build -t kj09/bankapp:latest ."
                    }    
                }
                
            }
        }
        
        stage('Trivy Image scan') {
            steps {
                sh "trivy image --format table -o image.html kj09/bankapp:latest"
            }
        }
        
        stage('Publish Docker Image') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cert') {
                    sh "docker push  kj09/bankapp:latest "
                    }    
                }
                
            }
        }
        
        stage('Deploy to K8s') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'devopsshack-cluster', contextName: '', credentialsId: 'k8s-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://8394D9215C2663ED930063636B228798.gr7.ap-south-1.eks.amazonaws.com') {
                    sh "kubectl apply -f ds.yml -n webapps"
                    sleep 30
                }
            }
        }
        
        stage('verify deployment') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'devopsshack-cluster', contextName: '', credentialsId: 'k8s-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://8394D9215C2663ED930063636B228798.gr7.ap-south-1.eks.amazonaws.com') {
                    sh "kubectl get svc -n webapps"
                    
                }
            }
        }
    }
}

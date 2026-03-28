pipeline {
    agent any
    
    tools {
        maven 'maven3'
    }
    
    environment {
        IMAGE_TAG = "v${BUILD_NUMBER}"
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', credentialsId: 'git', url: 'https://github.com/sandeshtamboli123/tummoc-assignment.git'
            }
        }
        
        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }
        
        stage('Testing') {
            steps {
                sh "mvn test"
            }
        }
        
        stage('Trivy FS Scan') {
            steps {
                sh "trivy fs --format table -o fs-report.html ."
            }
        }
        
        stage('Build') {
            steps {
                sh "mvn package"
            }
        }
        
        stage('Docker Image Build & Tag') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred') {
                        sh "docker build -t mynamesandesh/bankapp:$IMAGE_TAG ."
                    }
                }
            }
        }
        
        stage('Scan Image') {
            steps {
                sh "trivy image --format table -o image-report.html adijaiswal/bankapp:$IMAGE_TAG"
            }
        }
        
        stage('Push Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred') {
                        sh "docker push adijaiswal/bankapp:$IMAGE_TAG"
                    }
                }
            }
        }
        
        stage('Update Manifest File') {
            steps {
                        sh '''
                            sed -i "s|mynamesandesh/bankapp:.*|mynamesandesh/bankapp:${IMAGE_TAG}|" manifest.yaml
                            
                            # Deploy
                           kubectl apply -f manifest.yaml
                        '''
                    }
                }
            }
    }
}

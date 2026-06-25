pipeline {
    agent { label 'docker-maven-trivy' } 
    tools {
        maven 'maven3'
    }
    environment {
        SONAR_IP = '202.51.82.42'
    }
    stages {
        stage('Trivy FS Scan') {
            steps {
                sh 'trivy fs --exit-code 1 --severity HIGH,CRITICAL .'
            }
        }
        stage('Build and Sonar') {
            steps {
                withCredentials([string(credentialsId: 'sonarqube-token', variable: 'SECRET_TOKEN')]) {
                    sh 'mvn clean verify sonar:sonar \
                    -Dsonar.projectKey=cwvj-devsecops-demo \
                    -Dsonar.host.url="http://${SONAR_IP}:9000" \
                    -Dsonar.token="${SECRET_TOKEN}" \
                    -Dsonar.qualitygate.wait=true'
                }
            }
        }
        stage('GHCR Login') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'ghcr-creds', passwordVariable: 'GITHUB_TOKEN', usernameVariable: 'GITHUB_USER')]) {
                    sh '''
                        echo "$GITHUB_TOKEN" | docker login ghcr.io \
                            -u "$GITHUB_USER" \
                            --password-stdin
                    '''
                }
            }
        }
    }
}


pipeline {
    agent { label 'docker-maven-trivy' } 
    tools {
        maven 'maven3'
    }
    environment {
        SONAR_IP = '202.51.82.42'
        IMAGE_REPO = "ghcr.io/rishi-prajapati/cwvj-devsecops-demo"
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
        stage('Build Image') {
            steps {
                sh 'export DOCKER_BUILDKIT=0 && docker build --platform linux/amd64 -t "$IMAGE_REPO:$BUILD_NUMBER" -t "$IMAGE_REPO:latest" .'

            }
        }
        stage('Trivy Image Scan') {
            steps {
                sh 'trivy image --severity HIGH,CRITICAL "$IMAGE_REPO:$BUILD_NUMBER"'
            }
        }
        // stage('Push to GHCR') {
        //     steps {
        //         sh 'docker push "$IMAGE_REPO:$BUILD_NUMBER"'
        //         sh 'docker push "$IMAGE_REPO:latest"'
        //     }
        // }
    }
}


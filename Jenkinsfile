pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = 'dockerhub-creds'  // ID of Jenkins credential
        DOCKER_IMAGE = 'chalasaniakhil/python-app-demo' 
        // SONARQUBE_ENV = 'SonarQube'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/chalasaniakhil/Python_app_demo.git' 
            }
        }

        stage('SonarQube Scan') {
            steps {
                withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR_TOKEN')]) {
                    sh '''
                           /opt/sonar-scanner/bin/sonar-scanner \
                          -Dsonar.projectName=python-app-demo \
                          -Dsonar.projectKey=python-flask-app \
                          -Dsonar.sources=. \
                          -Dsonar.host.url=http://51.20.7.164:9000/ \
                          -Dsonar.login=$SONAR_TOKEN
                    '''
                }
            }
        }

        
        stage('Build Docker Image') {
            steps {
                script {                    
                    sh "docker build -t $DOCKER_IMAGE:${BUILD_NUMBER} ."                    
                }
            }
        }

        stage('Scan with Trivy') {
            steps {
                script {
                    sh """
                        trivy image --exit-code 0 --severity HIGH,CRITICAL --format table $DOCKER_IMAGE:$BUILD_NUMBER || true
                    """
                }
            }
        }

        stage('DockerHub Login') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    """
                }
            }
        }       

        stage('Push Image') {
            steps {               
                sh "docker push $DOCKER_IMAGE:${BUILD_NUMBER}"
                
            }
        }
    
        stage('Deploy to EC2') {
            steps {
                sshagent(['EC2_SSH_KEY']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ubuntu@16.170.247.74 '
                            docker pull chalasaniakhil/python-app-demo:$BUILD_NUMBER &&
                            docker stop flask-app || true &&
                            docker rm flask-app || true &&
                            docker run -d -p 5000:5000 --name flask-app chalasaniakhil/python-app-demo:$BUILD_NUMBER
                        '
                    """
                }
            }
        }
    }
    
}

pipeline {
    agent any
 
    tools {
        maven "Maven"
    }
 
    stages {
 
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/kausar32/spring-restbucks.git'
            }
        }
 
    
 
        stage('SonarQube Analysis') {
            steps {
                dir('server') {
                    withSonarQubeEnv('Sonar') {
                        sh '''
                        mvn org.sonarsource.scanner.maven:sonar-maven-plugin:5.7.0.6970:sonar \
                        -Dsonar.projectKey=Spring-RESTBucks
                        '''
                    }
                }
            }
        }
 
        stage('Docker Build') {
            steps {
                dir('server') {
                    sh '''
                    docker build -t kausar32/restbucks:latest .
                    '''
                }
            }
        }
 
        stage('Trivy Scan') {
            steps {
                dir('server') {
                    sh '''
                    trivy image \
                    --scanners vuln \
                    --pkg-types os \
                    --format table \
                    restbucks:latest
                    '''
                }
            }
        }
        stage('Push to Docker Hub') {
    steps {
        withCredentials([usernamePassword(
            credentialsId: 'dockerhub',
            usernameVariable: 'DOCKER_USER',
            passwordVariable: 'DOCKER_PASS'
        )]) {
 
            sh '''
            echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
            docker push kausar32/restbucks:latest
            docker logout
            '''
        }
    }
}
 
        stage('Deploy') {
            steps {
                sh '''
                docker stop restbucks-app || true
                docker rm restbucks-app || true
 
                docker run -d \
  --name restbucks-app \
  -p 8081:8080 \
  kausar32/restbucks:latest
 
                '''
            }
        }
    }
}

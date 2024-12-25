pipeline {
    agent any
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('git-checkout') {
            steps {
                git 'https://github.com/divyasatpute/Day-8-Project-8-secretsanta-generator.git'
            }
        }

        stage('Code-Compile') {
            steps {
               sh "mvn clean compile"
            }
        }

        stage('Unit Tests') {
            steps {
               sh "mvn test"
            }
        }

        stage('Trivy Dependency Scan') {
            steps {
                sh """
                    # Install Trivy if not already available
                    if ! [ -x \$(command -v trivy) ]; then
                        curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh -s -- -b /usr/local/bin
                    fi
                    
                    # Run Trivy scan on the project directory
                    trivy fs --severity HIGH,CRITICAL .
                """
            }
        }

        stage('Sonar Analysis') {
            steps {
               withSonarQubeEnv('sonar') {
                   sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Santa \
                   -Dsonar.java.binaries=. \
                   -Dsonar.projectKey=Santa'''
               }
            }
        }

        stage('Code-Build') {
            steps {
               sh "mvn clean package"
            }
        }

        stage('Docker Build') {
            steps {
               script {
                   withDockerRegistry(credentialsId: 'docker-cred') {
                       sh "docker build -t santa123 ."
                   }
               }
            }
        }

        stage('Docker Push') {
            steps {
               script {
                   withDockerRegistry(credentialsId: 'docker-cred') {
                       sh "docker tag santa123 divyasatpute/santa123:latest"
                       sh "docker push divyasatpute/santa123:latest"
                   }
               }
            }
        }

        stage('Docker Image Scan') {
            steps {
                sh """
                    # Run Trivy scan on the Docker image
                    trivy image divyasatpute/santa123:latest
                """
            }
        }
    }
}

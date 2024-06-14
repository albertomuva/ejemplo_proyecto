pipeline {
    agent any
    
    tools {
        maven 'maven-tool'
        jdk 'jdk17'
    }
    
    environment {
        SCANNER_HOME= tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/albertomuva/ejemplo_proyecto.git'
            }
        }
        
        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }
        
        stage('Unit Tests') {
            steps {
                sh "mvn test -DskipTests=true"
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube_Server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=EKART -Dsonar.projectName=EKART \
                    -Dsonar.java.binaries=. '''
                    
                }
            }
            
        }
        
        stage('OWASP DEpendency Check') {
            steps {
                dependencyCheck additionalArguments: ' --scan ./', odcInstallation: 'Dependency-check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        stage('Build') {
            steps {
                sh "mvn package -DskipTests=true"
            }
        }
        
        stage('Deploy To Nexus') {
            steps {
               withMaven(globalMavenSettingsConfig: 'settings-nexus', jdk: 'jdk17', maven: 'maven-tool', mavenSettingsConfig: '', traceability: true) {
                    sh "mvn deploy -DskipTests=true"
                }
                    
            }
        }
        
        stage('Build & Tag Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'dockerhub-user') {
                        
                        sh "docker build -t shopping-cart -f docker/Dockerfile ."
                        sh "docker tag  shopping-cart albertomv/shopping-cart:latest"
                    }
                }
            }
        }
        
        stage('Trivy Scan') {
            steps {
                sh "trivy image albertomv/shopping-cart:latest > trivy-report.txt "
                
            }
        }
        
        stage('Push the Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'dockerhub-user', toolName: 'docker') {
                        
                        sh "docker push albertomv/shopping-cart:latest"
                    }
                }
            }
        }
    }
}

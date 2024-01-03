Full Stack CICD DevOps Project | Virtual Browser DevOps project

steps
---------------------------------

launch ec2 - tx-large

install jenkins ,install docker , install sonarqube, install trivy

--> login into jenkins

	--> go to plugins
	--> sonarqube, OWASP Dependency-Check, docker --> install all the plugins


-------Pipeline script-----------


pipeline {
    agent any 
    
    environment {
        SCANNER_HOME = tool name: 'sonar-scanner', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
    }
    
    stages {
        stage('Git-Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/cloudtarun97/Vitual-Browser.git'
            }
        }
        
        stage('Owasp Dependency Check') {
            steps {
                script {
                    dependencyCheck additionalArguments: '--scan ./ --disableNodeAudit', odcInstallation: 'DC'
                    dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
                }
            }
        }
        
        stage('SonarQube') {
            steps {
                withSonarQubeEnv('sonar') {
                    script {
                        sh "$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=virtual-browser -Dsonar.ProjectName=virtual-browser -Dsonar.java.binaries=."
                    }
                }
            }
        }
        
        stage('Docker Build & Tag Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred') {
                        dir('/home/ubuntu/.jenkins/workspace/virtual-browser/.docker/firefox') {
                            sh "docker build -t cloudtarun97/vb:latest ."
                        }
                    }
                }
            }
        }

        stage('Trivy Docker scan') {
            steps {
                script {
                    sh "trivy image cloudtarun97/vb:latest > trivy.txt"
                }
            }
        }

        stage('Docker Push Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred') {
                        sh "docker push cloudtarun97/vb:latest"
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                sh "docker-compose up -d"
            }
        }
    }
}

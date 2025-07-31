pipeline{
    agent any

    environment{
        SCANNER_HOME= '/opt/sonar-scanner'
    }

    stages{
        stage('Clean Docker Container') {
            steps {
                script {
                    CONTAINER_ID = sh(
                        script: "docker ps -aqf 'name=webapp'",
                        returnStdout: true
                    ).trim()

                    if (CONTAINER_ID) {
                        echo "Removing container with ID: ${CONTAINER_ID}"
                        sh "docker stop ${CONTAINER_ID}"
                        sh "docker rm ${CONTAINER_ID}"
                    } else {
                        echo "Container is not setup"
                    }
                }
            }
        }

        stage('Git Checkout'){
            steps{
                git branch: 'master', changelog: false, poll: false, url: 'https://github.com/Nisha-Shah29/SpringBoot-WebApplication.git'
            }
        }
        stage('Code Compile'){
            steps{
                sh "mvn compile"
            }
        }
        stage('Run Test Cases'){
            steps{
                sh "mvn test"
            }
        }
        stage('Sonarqube Analysis') {
            steps {
                    withSonarQubeEnv('sonar-server') {
                        sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Java-WebApp \
                        -Dsonar.java.binaries=. \
                        -Dsonar.projectKey=Java-WebApp '''
    
                }
            }
        }
        stage('OWASP Dependency Check') {
            steps {
                   dependencyCheck additionalArguments: '--scan ./   ', odcInstallation: 'DP'
                   dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('Maven Build') {
            steps {
                    sh "mvn clean package"
            }
        }
        stage('Docker Build & Push') {
            steps {
                   script {
                       withDockerRegistry(credentialsId: '30ad250a-c88e-4b27-9dc6-285e0e71dfda') {
                            sh "docker build -t webapp ."
                            sh "docker tag webapp nisha2906/webapp:latest"
                            sh "docker push nisha2906/webapp:latest"
                        }
                   } 
            }
        }

        stage('Docker Image scan') {
            steps {
                    sh "trivy image nisha2906/webapp:latest"
            }
        }
        stage('Docker Container') {
            steps {
                   script {
                       withDockerRegistry(credentialsId: '30ad250a-c88e-4b27-9dc6-285e0e71dfda', toolName: 'docker') {
                            sh "docker run -d --name webapp -p 8080:8080 nisha2906/webapp:latest"
                        }
                   } 
            }
        }
    }
}
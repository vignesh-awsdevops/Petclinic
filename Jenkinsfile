pipeline {
    agent any 
    
    tools {
        jdk 'java11'
        maven 'maven'
    }
    
    environment {
        SONAR_SCANNER_HOME = "/opt/sonar-scanner/bin"
        PATH = "${SONAR_SCANNER_HOME}:${PATH}"
    }
    
    stages {
        
        stage("Git Checkout") {
            steps {
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/jaiswaladi246/Petclinic.git'
            }
        }
        
        stage("Compile") {
            steps {
                sh "mvn clean compile"
            }
        }
        
        stage("Test Cases") {
            steps {
                sh "mvn test"
            }
        }
        
        stage("Sonarqube Analysis") {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh ''' ${SONAR_SCANNER_HOME}/sonar-scanner \
                    -Dsonar.projectName=Petclinic \
                    -Dsonar.java.binaries=. \
                    -Dsonar.projectKey=Petclinic '''
                }
            }
        }
        
        stage("OWASP Dependency Check") {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --format HTML', odcInstallation: 'DP'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        stage("Build") {
            steps {
                sh "mvn clean install"
            }
        }
        
        stage("Docker Build & Push") {
            steps {
                script {
                    withDockerRegistry(credentialsId: '58be877c-9294-410e-98ee-6a959d73b352') {
                        sh "docker build -t adijaiswal/pet-clinic123:latest ."
                        sh "docker push adijaiswal/pet-clinic123:latest"
                    }
                }
            }
        }
        
        stage("TRIVY Security Scan") {
            steps {
                sh "trivy image adijaiswal/pet-clinic123:latest"
            }
        }
        
        stage("Deploy To Tomcat") {
            steps {
                sh "cp target/petclinic.war /opt/apache-tomcat-9.0.65/webapps/"
            }
        }
    }
}

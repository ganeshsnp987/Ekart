pipeline {
    agent any
    tools {
        maven 'maven3'
        jdk 'jdk17'
        dockerTool 'docker'
    }
    
    environment {
        SCANNER_HOME= tool 'sonar-scanner'
    }

    stages {
        stage('Git checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/ganeshsnp987/Ekart.git'
            }
        }
        
        stage('Compile Code') {
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
                withSonarQubeEnv('sonar') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=EKART -Dsonar.projectName=EKART -Dsonar.java.binaries=. '''
                    
    
}
            }
        }
        
        stage('OWASP DP Check') {
            steps {
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'DC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        stage('Build') {
            steps {
                sh "mvn package -DskipTests=true"
            }
        }
        
        stage('Artifact Store to Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'global-maven', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) 
                {
                sh "mvn deploy -DskipTests=true"
}
            }
        }
     
     stage('Build and Tag Docker image') {
            steps {
                withDockerRegistry([credentialsId: 'docker-cred', url: 'https://index.docker.io/v1/']) {
                sh "docker build -t ganeshsnp987/ekart:latest -f docker/Dockerfile ."
}
            }
        }
        
        stage('Trivy Scan') {
            steps {
                sh "trivy image ganeshsnp987/ekart:latest > trivy-report.txt"
            }
        }
        
        stage('Docker image push to Docker hub') {
            steps {
                withDockerRegistry([credentialsId: 'docker-cred', url: 'https://index.docker.io/v1/']) {
                sh "docker push ganeshsnp987/ekart:latest"
}
            }
        }
      stage('Deploy to k8s') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8s-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://172.31.11.4:6443') {
                sh "kubectl apply -f deploymentservice.yml -n webapps"
                sh "kubectl get svc -n webapps"
}
            }
        }  
        
    }
}

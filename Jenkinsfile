pipeline {
    agent any

    tools {
        maven 'maven3'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', credentialsId: 'git-cred', url: 'https://github.com/leroyjenkin/DevOps-MultiTierBankAppCI-pipeline.git'
            }
        }

        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }

        stage('Test') {
            steps {
                sh "mvn test -DskipTests=true"
            }
        }

        stage('File System Scan') {
            steps {
                sh "trivy fs --format table -o fs.html"
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                     sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=BG -Dsonar.projectName=BG -Dsonar.java.binaries=target'''
            }
    }
}

pipeline {
    agent any

    tools {
        maven 'maven3'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', credentialsId: 'git-cred', url: 'https://github.com/jaiswaladi246/Multi'
            }
        }

        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }

        stage('Hello') {
            steps {
                echo 'Hello World'
            }
        }

        stage('Hello') {
            steps {
                echo 'Hello World'
            }
        }
    }
}

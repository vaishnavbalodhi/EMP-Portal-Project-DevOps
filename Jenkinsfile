pipeline {
    agent any

    stages {
        stage('git checkout') {
            steps {
                checkout scmGit(branches: [[name: '*/feature_jenfile']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/vaishnavbalodhi/EMP-Portal-Project-DevOps.git']])
            }
        }
    }
    stage('Installing packages') {
            steps {
                git branch: 'feature_jenfile', url: 'https://github.com/vaishnavbalodhi/EMP-Portal-Project-DevOps.git'
                sh 'pip install -r requirements.txt'
                sh 'python3 app.py'
            }
        }
}

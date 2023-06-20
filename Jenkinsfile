pipeline {
    agent any

    stages {
        stage('git checkout') {
            steps {
                checkout scmGit(branches: [[name: '*/feature_jenfile']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/vaishnavbalodhi/EMP-Portal-Project-DevOps.git']])
            }
        }
        stage('Installing packages') {
            steps {
                sh 'sudo apt install python3-pip'
                sh 'pip install -r requirements.txt'
            }
         }
        stage('pylint testing') {
            steps {
                sh 'pip install pylint'
                sh 'pylint app.py'
            }
         }
    }
}

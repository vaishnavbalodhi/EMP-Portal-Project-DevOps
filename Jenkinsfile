pipeline {
        environment {
        scannerHome = tool 'sonar-scan'
    }
    agent any

    stages {
        stage('git checkout') {
            steps {
                checkout(
                    scmGit(
                        branches: [[name: '*/feature_jenfile']],
                        extensions: [],
                        userRemoteConfigs: [[url: 'https://github.com/vaishnavbalodhi/EMP-Portal-Project-DevOps.git']]
                    )
                )
            }
        }

        stage('Installing packages') {
            steps {
                sh 'sudo apt install python3-pip'
                sh 'pip install -r requirements.txt'
            }
        }

        stage('Static Code Checking') {
            steps {
                script {
                    // Run pylint on Python files and generate a report
                    sh 'find . -name \\*.py | xargs pylint -f parseable | tee pylint.log'
                    recordIssues(
                        tool: pyLint(pattern: 'pylint.log'),
                        unstableTotalHigh: 100
                    )
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    withSonarQubeEnv('sonar') {
                        // Run SonarQube scanner for code analysis
                        sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=DevOps-Project -Dsonar.sources=."
                    }
                }
            }
        }

        stage("Quality Gate"){
            timeout(time: 1, unit: 'HOURS') { // Just in case something goes wrong, pipeline will be killed after a timeout
            def qg = waitForQualityGate() // Reuse taskId previously collected by withSonarQubeEnv
            if (qg.status != 'OK') {
                error "Pipeline aborted due to quality gate failure: ${qg.status}"
            }
          }
        }
    }
}

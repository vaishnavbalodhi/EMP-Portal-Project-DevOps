pipeline {
    environment {
        scannerHome = tool 'sonar-scan'
    }
    agent any

    stages {
        stage('git checkout') {
            steps {
                checkout(
                    scm: [
                        $class: 'GitSCM',
                        branches: [[name: '*/feature_jenfile']],
                        extensions: [],
                        userRemoteConfigs: [[url: 'https://github.com/vaishnavbalodhi/EMP-Portal-Project-DevOps.git']]
                    ]
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
                    sh 'find . -name "*.py" | xargs pylint -f parseable | tee pylint.log'
                    recordIssues(
                        tools: [pyLint(pattern: 'pylint.log')],
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

       stage('SonarQube Quality Gates') {
            steps {
                script {
                    withSonarQubeEnv('sonar') {
                        timeout(time: 1, unit: 'HOURS') {
                            // Wait for SonarQube quality gates to pass/fail
                            def qg = waitForQualityGate()
                            if (qg.status != 'OK') {
                                error "Pipeline aborted due to quality gate failure: ${qg.status}"
                            }
                        }
                    }
                }
            }
        }
    }
}

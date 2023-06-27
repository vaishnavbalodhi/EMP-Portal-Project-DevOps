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
                // sh 'sudo apt install python3-pip'
                sh 'sudo su'
                sh 'pip install -r requirements.txt'
            }
         }
        // stage('Static code analysis'){
        // steps {
        //    sh 'pylint -f parseable --reports=no * > pylint.log' 
        // }
        // post {
        //     always {
        //         sh 'cat pylint.log'
        //         recordIssues healthy: 1, tools: [pyLint(name: 'report name', pattern: '**/pylint.log')], unhealthy: 2
        //         }
        //     }
        // }
        // stage('pylint testing') {
        //     steps {
        //         sh 'pip install pylint'
        //         sh 'pylint app.py'
        //     }
        //  }
        stage('SonarQube analysis') {
            steps{
                withSonarQubeEnv(credentialsId: 'sonarcred', installationName: 'sonar'){ 
                  sh "${scannerHome}/bin/sonar-scanner"
                }
            }
        }
    }
    post{
        always{
            script{
                def buildStatus = currentBuild.currentResult ?: 'UNKNOWN'
                def color = buildStatus== 'SUCCESS' ? 'good' : 'danger'

                slackSend(
                    channel: '#devops-project',
                    color: color,
                    message: "Build ${env.BUILD_NUMBER} ${buildStatus}: STAGE=${env.STAGE_NAME}",
                    teamDomain: 'xaidv05',
                    tokenCredentialId: 'slackcred'
                )
            }
        }
    }
}

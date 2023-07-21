pipeline {
    environment {
        scannerHome = tool 'sonar-scan'
        registry = "vaishnav1/devops_project"
        registryCredential = 'dockerhub'
        dockerImage =''
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

       //  stage('SonarQube Analysis') {
       //      steps {
       //          script {
       //              withSonarQubeEnv('sonar') {
       //                  // Run SonarQube scanner for code analysis
       //                  sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=DevOps-Project -Dsonar.sources=."
       //              }
       //          }
       //      }
       //  }

       // stage('SonarQube Quality Gates') {
       //      steps {
       //          script {
       //              withSonarQubeEnv('sonar') {
       //                  timeout(time: 1, unit: 'HOURS') {
       //                      // Wait for SonarQube quality gates to pass/fail
       //                      def qg = waitForQualityGate()
       //                      if (qg.status != 'OK') {
       //                          error "Pipeline aborted due to quality gate failure: ${qg.status}"
       //                      }
       //                  }
       //              }
       //          }
       //      }
       //  }
        stage ('Clean Up') {
            steps {
                // Stop and remove Docker containers
                sh returnStatus: true, script: 'docker stop $(docker ps -a | grep ${JOB_NAME} | awk \'{print $1}\')'
                sh returnStatus: true, script: 'docker rmi $(docker images | grep ${registry} | awk \'{print $3}\') --force'
                sh returnStatus: true, script: 'docker rm -f ${JOB_NAME}'
            }
        }
        stage('Build Image') {
            steps {
                script {
                    // Build Docker image with a unique tag
                    img = registry + ":${env.BUILD_ID}"
                    println("${img}")
                    dockerImage = docker.build("${img}")
                }
            }
        }
        stage('Push To DockerHub') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', registryCredential) {
                        // Push Docker image to DockerHub
                        dockerImage.push()
                    }
                }
            }
        }
        stage('Deploy to containers') {
            steps {
                // Deploy Docker image to containers
                sh label: '', script: "docker run -d --name ${JOB_NAME} -p 5002:5000 ${img}"
            }
        }
        // stage('Deploy to Kubernetes EKS') {
        //     steps {
        //         script {
        //             // def clusterName = "eks-deployment-cluster"
                    
        //             withKubeConfig(caCertificate: '', clusterName: 'eks-deployment-cluster', contextName: '', credentialsId: 'kubeconfig', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
        //                 // sh "sed -i 's|\${ENV_IMAGE}|${img}|g' deployment.yaml" // Replace placeholder with Docker image name in deployment.yaml
        //                 sh "kubectl apply -f deployment.yaml" // Apply deployment configuration
        //                 sh "kubectl apply -f service.yaml" // Apply service configuration
        //             }
        //         }
        //     }
        // }
        stage('Deploy to Kubernetes AKS') {
            steps {
                script {
                    // Retrieve the selected target cluster
                    def kubeconfig

                    withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                        // sh "sed -i 's|\${ENV_IMAGE}|${img}|g' deployment.yaml" // Replace placeholder with Docker image name in deployment.yaml
                        sh "kubectl apply -f deployment.yaml --kubeconfig=$KUBECONFIG" // Apply deployment configuration
                        sh "kubectl apply -f service.yaml --kubeconfig=$KUBECONFIG" // Apply service configuration
                    }
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

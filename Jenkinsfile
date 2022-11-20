pipeline {
    agent any
    environment {
        VERSION = "${env.BUILD_ID}"
    }
    stages{
        stage('sonar quality analysis') {
            agent {
                docker {
                    image 'maven'
                }
            }
            steps {
                script {
                        withSonarQubeEnv(credentialsId: 'sonar-token') {
                        sh 'mvn clean package sonar:sonar'
                    }
                }

            }
        }

        stage ('Quality gate status') {
            steps {
                script{
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
            }
        }

        stage ('docker build and push nexus'){
            steps{
                script{
                    withCredentials([string(credentialsId: 'nexus_cred', variable: 'nexus_creds')]) {
                    
                    sh '''
                        docker build -t 54.196.216.223:8083/springapp:${VERSION} .
                        docker login -u admin -p $nexus_creds 54.196.216.223:8083
                        docker push 54.196.216.223:8083/springapp:${VERSION}
                        docker rmi 54.196.216.223:8083/springapp:${VERSION}
                    '''
                    }
                }
            }
        }
    }
}

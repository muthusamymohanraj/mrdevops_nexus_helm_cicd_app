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
        stage('Identifying misconfigs using datatree in hlem charts'){
                steps {
                    script {
                        dir('kubernetes/myapp') {
                            withEnv(['DATREE_TOKEN=3e5edcc3-aad3-486f-a96c-c3a7def9626c']) {
                            sh 'helm datree test .'
                            }


                        }
                    }
                }
            
        }

        stage ('pushing helm to nexus'){
            steps {
                script{
                    withCredentials([string(credentialsId: 'nexus_cred', variable: 'nexus_creds')]) {
                        dir('kubernetes/') {
                        sh '''
                            helmversion=$(helm show chart myapp | grep version | cut -d: -f 2 | tr -d ' ')
                            tar -zxvf myapp-${helmversion}.tgz myapp/
                            curl -u admin:$nexus_creds http://54.196.216.223:8081/repository/helm-host/ --upload-file myapp-${helmversion}.tgz -v

                            '''
                        }
                    }

                }
            }
        }
    }
    
    post {
		always {
			mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> URL de build: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "${currentBuild.result} CI: Project name -> ${env.JOB_NAME}", to: "logintomohanraj@gmail.com";  
		}
	}
}

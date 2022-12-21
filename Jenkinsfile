pipeline{
    agent any
    environment{
        VERSION = "${env.BUILD_ID}"
    }
    stages{
        stage('Sonar quality check'){
            agent {
                docker{
                    image 'maven'
                }
            }
            steps{
                script{
                    withSonarQubeEnv(credentialsId: 'sonar-token') {
                    
                    sh 'mvn clean package sonar:sonar'

                    }
                }
            }
        }
        stage('Quality Gate status'){
            steps{
                script{
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
            }
        }
        stage('docker build & docker push to Nexus repo'){
            steps{
                script{
                    withCredentials([string(credentialsId: 'nexus_passwd', variable: 'nexus-creds')]){
                        sh '''
                        docker build -t 172.31.12.32:8083/springapp:${VERSION} .
                        docker login -u admin -p nexus 172.31.12.32:8083
                        docker push 172.31.12.32:8083/springapp:${VERSION}
                        docker rmi 172.31.12.32:8083/springapp:${VERSION}
                        '''
                    }
                }
            }
        }
	stage('Identifying misconfigs using datree in helm charts'){
		dir('kubernetes/myapp/') {
			sh 'helm datree test .'
		}
	}
    }
    post {
		always {
			mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> URL de build: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "${currentBuild.result} CI: Project name -> ${env.JOB_NAME}", to: "avkkumaran@gmail.com";  
		}
	}
}

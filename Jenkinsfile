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
                    docker login -u admin -p $nexus-creds 172.31.12.32:8083
                    docker push 172.31.12.32:8083/springapp:${VERSION}
                    docker rmi 172.31.12.32:8083/springapp:${VERSION}
                    '''
                    }
                }
            }
        }
	stage('Identifying misconfigs using datree in helm charts'){
	    steps{
		script{
		    dir('kubernetes/myapp/') {
			withEnv(['DATREE_TOKEN=68bfb63a-426c-45a9-98cf-bc83da24c0fb']) {
			sh 'helm datree test .'
			}
		    }
		}
	    }
	}
	stage{
	    steps{
		script{
		    withCredentials([string(credentialsId: 'nexus_passwd', variable: 'nexus-creds')]){
			dir('kubernetes/') {
		    	sh '''
		    	helmversion=$(helm show chart myapp | grep version | cut -d: -f 2 | tr -d ' ')
		    	tar -czvf myapp-${helmversion myapp/
		    	curl -u admin:$nexus-creds http://172.31.12.32:8081/repository/helm-repo/ --upload-file myapp-${helmversion}.tgz -v
		    	'''
			}
		    }
		}
	    }
	}
    }
    post {
		always {
			mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> URL de build: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "${currentBuild.result} CI: Project name -> ${env.JOB_NAME}", to: "avkkumaran@gmail.com";  
		}
    }
}

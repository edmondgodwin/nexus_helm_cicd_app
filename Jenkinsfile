pipeline{
    agent any
    environment {

        VERSION = "${env.BUILD_ID}"
    }

    stages{

        stage('sonar static code analysis checks'){

            agent{

                docker {
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

        stage('Quality Gate Status'){

            steps{

                script{

                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-new-token'
                }
            }
        }

        stage('docker build and push artifact to nexus repo'){

            steps{

                script{
                    withCredentials([string(credentialsId: 'nexus-passwd', variable: 'nexus_creds')]) {
                        sh '''
                            docker build -t 44.197.115.178:8083/springapp:${VERSION} .

                            docker login -u admin -p $nexus_creds 44.197.115.178:8083

                            docker push 44.197.115.178:8083/springapp:${VERSION}

                            docker rmi 44.197.115.178:8083/springapp:${VERSION}

                         '''

                    }
                    
                }
            }
        }
        stage('Identifying misconfigurations usind datree in helm charts'){

            steps{
                script{
                    dir('kubernetes/myapp/') {
                            withEnv(['DATREE_TOKEN=e42e9fea-984a-43ad-81b3-1d51e226203f']) {
                        sh 'helm datree test .'
                        }
                    }
                }
            }
        }
        stage('Pushing the helm charts tp helm repo'){

            steps{
                script{
                    withCredentials([string(credentialsId: 'nexus-passwd', variable: 'nexus_creds')]) {
                      dir('kubernetes/') {
                        sh ''' 
                        helmversion-$(helm show chart myapp | grep version | cut -d: -f 2 | tr -d '  ')
                        tar -czvf myapp-${helmversion}.tgz myapp/
                        curl -u admin:$nexus_creds http://44.197.115.178:8081/repository/helm-repo/ --upload-file myapp-${helmversion}.tgz -v'
                        '''
                      
                      }
                }
            }
        }
    }
    post {
		always {
			mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> URL de build: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "${currentBuild.result} CI: Project name -> ${env.JOB_NAME}", to: "thehakaiva@gmail.com";  
		}
	}
}
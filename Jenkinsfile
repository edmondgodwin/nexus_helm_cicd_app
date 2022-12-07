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
    }
}
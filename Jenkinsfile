def registry = 'https://himam29.jfrog.io/'
pipeline {
	agent any
    tools {
        maven "MyMaven"
    }
	environment {
		SCANNER_HOME= tool 'sonar-scanner'
        }
        stages {
            stage ('Checkout from git'){
                steps {
                    git branch: 'main', url: 'https://github.com/himam29/springbootapp.git'
            }
        }
        stage ('Maven Build'){
                steps {
                   sh 'mvn clean install'
            } 
        }
        stage ('Unit Test'){
            steps {
                echo '<----------------------Unit Test Under Progess------------------>'
                sh 'mvn surefire-report:report'
                echo '<----------------------Unit Test Done------------------>'
            }
        }
		
	stage ('Sonarqube analysis'){
            steps {
                script {
                    withSonarQubeEnv('sonarserver') {
                         sh  ''' ${SCANNER_HOME}/bin/sonar-scanner -Dsonar.projectName=springbootapp \
                         -Dsonar.projectKey=springbootapp '''
			    
			// sh '${SCANNER_HOME}/bin/sonar-scanner --version'
                    }
                }
            }
        }
        stage ('Quality Gate Test'){
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'SonarqubeServer'
                }
            }
        }
        stage ('Build Docker Image'){
            steps {
                script {
                    sh 'docker build -t myrepo .'
                }
            }
        }
	stage("Jar Publish") {
            steps {
                script {
                        echo '<--------------- Jar Publish Started --------------->'
                         def server = Artifactory.newServer url:registry+"/artifactory" ,  credentialsId:"jfrog"
                         def properties = "buildid=${env.BUILD_ID},commitid=${GIT_COMMIT}";
                         def uploadSpec = """{
                              "files": [
                                {
                                  "pattern": "target/springbootApp.jar",
                                  "target": "springbootapprelease-libs-release-local",
                                  "flat": "false",
                                  "props" : "${properties}",
                                  "exclusions": [ "*.sha1", "*.md5"]
                                }
                             ]
                         }"""
                         def buildInfo = server.upload(uploadSpec)
                         buildInfo.env.collect()
                         server.publishBuildInfo(buildInfo)
                         echo '<--------------- Jar Publish Ended --------------->'  
                
                }
            }   
        }
		
      stage ("Push Image to ECR")  {
        steps {
            script {
	      withAWS(credentials: "AWSCLI", region: 'us-east-2') {	    
                sh 'aws ecr get-login-password --region us-east-2 | docker login --username AWS --password-stdin 146869492184.dkr.ecr.us-east-2.amazonaws.com'
                //sh 'docker build -t ncplrepo .'
                sh 'docker tag myrepo:latest 146869492184.dkr.ecr.us-east-2.amazonaws.com/springbootapp:latest'
                sh 'docker push 146869492184.dkr.ecr.us-east-2.amazonaws.com/springbootapp:latest'
	      }
              }
           }
       }
   }
}

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
	}
}

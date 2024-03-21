pipeline {
	agent any
    tools {
        maven "MyMaven"
    }
	environment {
		SCANNER_HOME= tool 'SonarQubeScanner'
        }
        stages {
            stage ('Checkout from git'){
                steps {
                    git branch: 'main', url: 'https://github.com/bkrrajmali/springbootapp.git'
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
                    withSonarQubeEnv('SonarServer') {
                        sh  ''' ${SCANNER_HOME}/bin/SonarQubeScanner -Dsonar.projectName=springbootapp \
                        -Dsonar.projectKey=springbootapp '''
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

pipeline {
    tools {
        maven "Maven3"
    }
    agent any
        environment {
            SCANNER_HOME= tool 'sonar-scanner'
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
                    withSonarQubeEnv('sonar-server') {
                        sh  ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=springbootapp \
                        -Dsonar.projectKey=springbootapp '''
                    }
                }
            }
        }
        stage ('Quality Gate Test'){
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar'
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
                         def server = Artifactory.newServer url:registry+"/artifactory" ,  credentialsId:"jfrogaccess"
                         def properties = "buildid=${env.BUILD_ID},commitid=${GIT_COMMIT}";
                         def uploadSpec = """{
                              "files": [
                                {
                                  "pattern": "target/springbootApp.jar",
                                  "target": "ncplmaven-libs-release-local",
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
                sh 'aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 037500517393.dkr.ecr.us-east-1.amazonaws.com'
                //sh 'docker build -t ncplrepo .'
                sh 'docker tag myrepo:latest 037500517393.dkr.ecr.us-east-1.amazonaws.com/ncplrepo:latest'
                sh 'docker push 037500517393.dkr.ecr.us-east-1.amazonaws.com/ncplrepo:latest'

            }
        }
    }
    stage('Kuberernetes Deployment'){
        steps{
            script {
                sh 'aws eks --region us-east-1 update-kubeconfig --name my-first-eks-cluster'
                sh 'kubectl apply -f eks-deploy-k8s.yaml'
            }
        }
    }    
    }
}

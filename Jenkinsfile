def registry = 'https://edvindza.jfrog.io'
pipeline {
  agent any
  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/eddzaa/newncplspringbootapp.git'
      }
    }
 stage('Pull Changes') {
      steps {
        sh 'git pull origin main'
        echo "WebHook Is Successfull Change"
      }
      }

stage ('Build') {
          steps {

            sh 'mvn clean install'    

            }
      }

   stage('Unit Test') {
      steps {
        echo '<--------------- Unit Testing started  --------------->'
        sh 'mvn surefire-report:report'
        echo '<------------- Unit Testing stopped  --------------->'
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
                                  "target": "maven-libs-release/{1}",
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
  stage('Build Docker Image') {
            steps {
                script {
                    // Build the Docker image
                    sh 'docker build -t myrepo .'
                }
            }
    }

    // Uploading Docker images into AWS ECR
    stage('Pushing to ECR') {
     steps{  
         script {
                sh 'aws ecr get-login-password --region us-east-2 | docker login --username AWS --password-stdin 656952365822.dkr.ecr.us-east-2.amazonaws.com'
                sh 'docker tag myrepo:latest 656952365822.dkr.ecr.us-east-2.amazonaws.com/myrepo:latest'
                sh 'docker push 656952365822.dkr.ecr.us-east-2.amazonaws.com/myrepo:latest'
         }
        }
      }

   stage('Deploy to EKS') {
            steps {
                script {
                    // Authenticate with the EKS cluster (ensure AWS credentials are configured)
                    sh 'aws eks --region us-east-2 update-kubeconfig --name prod'
                    
                    // Apply Kubernetes manifest files to deploy your application
                     // sh "kubectl delete -f eks-deploy-k8s.yaml"
                      sh "/var/lib/jenkins/bin/kubectl apply -f eks-deploy-k8s.yaml"
                }
            }
        }

  }
}

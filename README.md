pipeline {

  environment {
    dockerimagename = "thetips4you/nodeapp"
    dockerImage = ""                              ## here i specified which docker image i will use
  }

  agent any

  stages {

    stage('Checkout Source') {
      steps {
        git 'https://github.com/shazforiot/nodeapp_test.git'     ## I entered the repository part of the image I want to import here
      }
    }

    stage('Build image') {
      steps{
        script {
          dockerImage = docker.build dockerimagename    ## After pulling the image from the repository, I will build it and push it to dockerhub in the next step.
        }
      }
    }

    stage('Pushing Image') {
      environment {
               registryCredential = 'dockerhublogin'   ## We provide docker hub information to log in to the account.
           }
      steps{
        script {
          docker.withRegistry( 'https://registry.hub.docker.com', registryCredential ) {   ## After logging in, I specified which version to tag the image as
            dockerImage.push("latest")
          }
        }
      }
    }

    stage('Deploying App to Kubernetes') {
      steps {
        script {
          kubernetesDeploy(configs: "deploymentservice.yml", kubeconfigId: "kubernetes")
        }
      }
    }

  }

}

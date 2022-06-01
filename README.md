###Introduction:

###Dockerhub Credential:I gave dockerhub information from manage credential for login

###Kubernetes Credential:first we got the information to edit the file with "cat config" in the /.minikube file.then we got the certificate information with "cat /.minikube/profiles/minikube/ca.crt".after that we got the key from "cat /.minikube/profiles/minikube/clien.key".after that we got the certificate information from "cat /.minikube/profiles/minikube/client.crt".we rearranged and saved the conf file that we got in step 1 with this information and gave it to jenkins credential(kubernetes configuration(kubeconfig)) a enter directly.















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
            dockerImage.tag([“myfirstimage”,”latest”])
            dockerImage.push("latest")
          }
        }
      }
    }

    stage('Deploying App to Kubernetes') {
      steps {
        script {
          kubernetesDeploy(configs: "deploymentservice.yml", kubeconfigId: "kubernetes")  ## in this step we deployed to kubernetes
        }
      }
    }

  }

}

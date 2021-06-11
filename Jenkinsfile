#!/usr/bin/env groovy

pipeline {
	timeout(time: 30, unit: "MINUTES")
    agent { label 'EKS-Build-Deploy' }

    options {
        timestamps()
    }

    environment {
        BRANCH_NAME = $(git symbolic-ref HEAD 2>/dev/null) || (unnamed branch)    // detached HEAD
        BRANCH_NAME = ${branch_name##refs/heads/}

        SHA_ID = $(git rev-parse HEAD)
        //  if the commit is pointed to by a tag, check the output of:
        GIT_TAG = $(git describe --exact-match ${SHA_ID})
        SHORT_SHA = $(git rev-parse HEAD | cut -c 1-8)
	}

    stages {
    
        /*  build an image from a Dockerfile and tag it.
            set the docker build tag name dynamically -- if build was triggered by a git tag, use that as the docker tag,
            otherwise use the first 8 characters of the SHA.
         */
        stage('Build') {
            steps {
                 checkout scm
                 // Jenkins configured as url: 'https://github.com/rb-devops/spring-boot-app.git'

		 SHORT_SHA = $(git rev-parse HEAD | cut -c 1-8)

		 // check for tag value
		 GIT_TAG = $(git describe --exact-match ${SHA_ID})

		 dockerfile {
                      filename 'Dockerfile'
                      label ${SHORT_SHA}
                      additionalBuildArgs  '--build-arg version=${SHORT_SHA}'
                 }

		 sh "docker build -t ${fullImageName}:${SHORT_SHA} ."
			
		 sh 'docker tag ${THE_TAG} aws_account_id.dkr.ecr.region.amazonaws.com/spring-boot-app'

            }
	    steps {
		when {
		    expression { ${GIT_TAG} != '' }
		    echo 'tag the image'
		    sh 'docker tag ${IMAGE} ${IMAGE}:${GIT_TAG}' 
		}
            }

	}

        /* authenticate to AWS and push the above image to an application specific ECR repository.
           determine where to upload the docker image -- if image tag is based of git tag, push it to us-east-1 AND us-west-2 regions of AWS. 
           otherwise (SHA) just push to us-east-1. 
         */
        stage('Publish') {
            steps {
               echo 'Pushing image to registry...'
               sh 'aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin spring-boot-app'
               sh 'docker push aws_account_id.dkr.ecr.region.amazonaws.com/spring-boot-app' 

            }
            steps {        
	         when {
                 // only if tagged
                    expression { env.GIT_TAG != '' }
		    echo 'Pushing tagged image to registry...'
		    sh 'aws ecr get-login-password --region us-west-2 | docker login --username AWS --password-stdin spring-boot-app'
                    sh 'docker push aws_account_id.dkr.ecr.region.amazonaws.com/spring-boot-app' 
               }
           }
        }

        /*  Deploy the updated image from previous two steps to a lower EKS environment. 
            This can be with kubectl apply or other methods you are comfortable with.

		   https://docs.aws.amazon.com/eks/latest/userguide/sample-deployment.html

           Provide logic to automatically deploy the image if its
           one of the following branches:  development, staging, or uat.   
           AND tagged with the git sha ( as opposed to a git tag )
	 */
        stage('Deploy') {
	     when {
             // only if one of these branches and assume there is a k8s deployment.yaml file included in the application repository.
	          expression { env.BRANCH_NAME ==~ /(development|staging|uat)/ && env.GIT_TAG != '')/ }
            
	         steps  {
                      sh 'kubectl create -f deployment.yml'
                }
             }

           }

	}    // end stages

	post {
            failure {
            // notify users when the Pipeline fails
            mail to: 'NBC-Devops-team@nbc.com',
                 subject: "Failed Pipeline: ${currentBuild.fullDisplayName}",
                 body: "Something went wrong with ${env.BUILD_URL}"
        }
	    success {
            // notify users when the Pipeline succeeds
            mail to: 'NBC-Devops-team@nbc.com',
                 subject: "Successful Pipeline: ${currentBuild.fullDisplayName}",
                 body: "Latest build ${env.BUILD_URL}"
        }
    }

}

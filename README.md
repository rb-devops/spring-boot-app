# spring-boot-app

Create a Jenkins job using Declarative Pipelines

 1.  Create a build stage in your jenkins job.
        - Show the steps for how you would build an image from a Dockerfile and tag it.
        - Include logic to determine, and set, the docker build tag name dynamically. 
	   if the build was triggered by a git tag then use that as the docker tag,
		otherwise use the first 8 characters of the SHA.

  2.  Create a publish stage that pushes above image to a container registry.
        - Provide steps/logic for how you would authenticate to AWS and push the above image to an application specific ECR repository.
        - Include conditional logic to determine where to upload the docker image. 
            Specifically if the docker image tag is based off a git tag push it to us-east-1 AND us-west-2 regions in AWS. 
                 otherwise just push to us-east-1.

  3.  Create a stage that conditionally deploys to a lower EKS environment.

        - Deploy the updated image from previous two steps to a lower EKS environment. 

        - This can be with kubectl apply or other methods you are comfortable with.

		https://docs.aws.amazon.com/eks/latest/userguide/sample-deployment.html

        - Provide logic to automatically deploy the image if its
            one of the following branches: development, staging, or uat.
            AND tagged with the git sha ( as opposed to a git tag )
   
	-->  Assume there is a k8s deployment.yaml file included in the application repository.

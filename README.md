# CI/CD Pipeline on AWS
This project demonstrates the process of creating a Continuous Integration and Continuous Deployment (CI/CD) Pipeline using the following services on AWS:
- Amazon ECS: Containerized application deployment and management
- Amazon ECR: Storing Docker Images
- AWS CodePipeline: Orchestrating the CI/CD Process
- AWS CodeBuild: Building and Testing Code

## Prerequisites
Before building the pipeline, these were the first steps required:
- Creating an active AWS account
- Setting up IAM User with permissions to ECS, ECR, CodePipeline, and CodeBuild
- Installing and Configuring the AWS CLI
- Docker Desktop installed

## Cloning 2048 Game Files 
Since this project is focused on implementing a CI/CD pipeline on AWS, the code used on this project was cloned from an existing Github repository which also contains a **Dockerfile** to containerize the 2048 game present in the directory. 
> `git clone https://github.com/techwithlucy/ztc-projects.git`

- The Dockerfile creates a lightweight container for hosting the 2048 game using the Nginx web server. The file also transfers all of the 2048 game files into the Nginx's default web root directory and exposes port 80, ensuring they are ready to be served and accessible over HTTP.
- Finally, the Dockerfile contains a `CMD` command that ensures the Nginx server runs in the foreground when the container starts, serving the 2048 game seamlessly.

## Run Docker Container and Push to ECR
1. Create an ECR Repository
  - Go to the Amazon ECR Console
  - Click on Create Repository
  - Obtain the repository URI for later use
2. Build Docker Image
```
docker build -t 2048-game .
```
3. Tag Docker Image for ECR
```
docker tag 2048-game:latest (ECR_URI):latest
```
4. Authenticate with ECR
```
aws ecr get-login-password --region (AWS_REGION) | docker login --username AWS --password-stdin (ECR_URI)
```
5. Push Docker Image to ECR
```
docker push (ECR_URI):latest
```

## Set Up Amazon ECS
1. Create an ECS Cluster
   - Go to ECS Cluster
   - Click on **Create Cluster** and follow the console wizrard to create a cluster
   - Using AWS Fargate as the Cluster Infrastructure
2. Create a Task Definition
   - In the ECS Console, click on **Task Definition** and then on **Create New Task Definition**
   - Select **Fargate** as Launch Type
   - Specify Container details such as name, image using **ECR_URI**, and port mapping using **Port 80**
3. Create a Service
   - Under the created Cluster select the Service tab and click create
   - Configure the Service, selecting launch type as **Fargate**, task definition as the **Recently Created Task Definition**, and number of tasks as **1**

## Set Up AWS CodePipeline
1. Create a CodePipeline
   - Go to CodePipeline console
   - Click on **Create Pipeline**
   - Name the pipeline and choose a new service role or an exisiting one
2. Configure **Source Stage**
   - For Source provider, select GitHub
   - Connect to GitHub through OAuth app in AWS Console
   - Create a **Repository** in **GitHub** using the **Cloned Website Code** for the 2048 game
   - Select the created repository and the main branch
3. Configure **Build Stage**
   - Select **Add Build Stage**
   - Choose **AWS CodeBuild**
   - Create CodeBuild Proejct
       - Source Provider: GitHub and connect to project repository
       - Enviorment: Keep Configuration as default
       - Serivce Role: Create an IAM Role
           - Attach the following Manager Policies
               - AmazonEC2ContainerRegistryFullAccess
               - AWSCodeBuildDeveloperAccess
               - AmazonS3FullAccess
               - Attach Inline Policy for ECS
               - ```
                 {
                   "Version": "2012-10-17",
                   "Statement": [
                     {
                       "Effect": "Allow",
                       "Action": [
                         "ecs:UpdateService",
                         "ecs:DescribeServices"
                       ],
                       "Resource": "(ENTER_YOUR_ECS_SERVICE_ARN)"
                     }
                   ]
                 }
                 ```
       - Buildspec Configuration: Click on **Use a buildspec file**
           - The buildspec.yml file is included in the website code
           - Modify the buildspec.yml file to reflect the project's ECR URI and ECS service details
             ```
             version: 0.2
             
             phases:
                 pre_build:
                     commands:
                         - echo Logging in to Amazon ECR...
                         - aws ecr get-login-password --region <your-region> | docker login --username AWS --password-stdin <your-ecr-repository-uri>
                 build:
                     commands:
                         - echo Building the Docker image...
                         - docker build -t <image-name> .
                         - echo Tagging the Docker image...
                         - docker tag <image-name>:latest <your-ecr-repository-uri>:latest
                 post_build:
                     commands:
                         - echo Pushing the Docker image to Amazon ECR...
                         - docker push <your-ecr-repository-uri>:latest
                         - echo Creating imagedefinitions.json file for ECS deployment...
                         - echo '[{"name":"<container-name>","imageUri":"<your-ecr-repository-uri>/<repository-name>:latest"}]' > imagedefinitions.json

             artifacts:
                 files:
                     - imagedefinitions.json
             ```
       - Artifact Configuration: Select Amazon S3 and a select an S3 bucket or create one for CodeBuild Artifacts
4. Configure **Deploy Stage**
   - Click **Add Deploy Stage**
   - Select Amazon ECS as the deploy provider
   - Choose the ECS Cluster and service that was recently created
5. Review and Create
   - Review the pipeline configuration and click **Create Pipeline**

## Verify the Deployment
1. Check CodePipeline
   - Go to CodePipeline Console
   - Check the pipeline execution status.
   - If everything is set up correctly, the pipeline should progress from source --> build --> deploy
2. Verify ECS Service
   - Go to ECS Console
   - Verify that the ECS service was updated and is running the new Docker image
3. Test the Application
   - Access the ECS service via the direct IP
   - Get the IP address of the website by navigating to ECS > Cluster > (CREATED_ECS_CLUSTER_FOR_2048_GAME)
   - Under the Tasks tab, select the latest task
   - Under the Networking tab, you will find the Public IP address
   - Type in the Website URL:
     ```
     http://(PUBLIC_IP_ADDRESS):80
     ```

## Conclusion
With the whole project setup, there should be a complete CI/CD pipeline that automatically deploys a containerized application whenever code changes are committed and pushed to GitHub. The AWS CodePipeline uses Amazon ECS, ECR, AWS CodePipeline, and AWS CodeBuild, as this setup will automate the process of building, testing, and deploying this application to ECS whenever code changes are pushed to the repository for simplified setup of the application. 

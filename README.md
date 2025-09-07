# CI/CD Pipeline on AWS
This project is focused on building a complete **CI/CD pipeline on AWS** to automatically deploy a containerized 
application whenever code changes are committed. This pipeline uses **AWS CodePipeline** and **AWS CodeBuild** to 
automate the process of building, testing, and deploying code. **Amazon ECS** is also used to containerize and
deploy the application within the pipeline.

## Building Docker Image
> The Code used for the 2048 game that was used for the CI/CD pipeline was cloned from a existing repository.
> 
> `git clone https://github.com/techwithlucy/ztc-projects.git`
- The 2048 Code contains a Dockerfile which will define the steps to run the container.
- The Dockerfile is designed to create a lightweight container for hosting the 2048 game using the Nginx web server.

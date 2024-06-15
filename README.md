# aws-ecs
## Cluster
 - A logical grouping of your ECS tasks, service & other resources in an AWS region.
## EC2 Launch Type
 - You run your taks on the EC2 instances you manage that are associated with your ECS Cluster. they are called container instances.
 - container instances must run be running the ECS agent & docker.
 - AWS provides ECS-optimize AMI's for container instances.
 - You need to manage the scaling of your container instances to ensure there is enough capacity for new tasks.
 - All management tasks such as patching, ECS agent version upgrades, and secutity of the container instances are your responsibility.
 - serverless way to deploy container on AWS.
 - you focus on building & deploying your containerized applications.
## Capacity Providers
- A newer and enhanced way to manage the infrastucture your tasks on.
- capacity provider strategies with one or more capacity providers determine how the tasks are run.
- An ECS cluster can have one or more capacity providers and an optional default capacity provider strategy.
- For Amazon ECS workloads on AWS Fargate , FARGATE and FARGET_SPOT capacity providers are added to your ECS cluster on creation.
- On autoscaling Group capacity providers, ECS can mange the scaling of the container instance capacity. But you still mange the rest.
- You can include one or more capacity providers in a strategy and attach it to your ECS service or standalone tasks.
## AWS Farget Launch Type
- The infrastucture's security, scalability, and reliability are AWS's responsibility.
## AWS ECS
- task defination
- services
## Task Defination 
- **TASK DEfination** 
    - is json text file describing the properties of your containers that will rum on ECS, such as image,port,CPU, memory,  IMA role, logging etc.
    - It can contain up to 10 containers, but it shoud define multiple containers only if they have to deploy together.
    - standalone tasks using taskdefination file 
    - You can run tasks at specific intervals or in response to an event with Amazon EventBridge.
- **Services**
    - It allow you to maintain a desired number of tasks running.
    - If any of the tasks fail, the Amazon ECS service scheduler creates a replacement tasks to keep the number.
    - You can automatically scale the number of your ECS tasks in your service with Application Auto Scaling.
    - You can attach Application Load Blancer to your ECS services & distribute to your ECS tasks.

# Envirnoment Variables on AWS CodeBuild
- plaintext
- AWS System Manager paramerter Store
    - supports encryped & unencrypted parameters.
    - Free for standard (Max. 4kb) parameters.
- AWS Secrets Manger
    - your secrets are automatically encrypted. 
    - Supports automatically secret rotation with integraed BDs like Amazone RDS.
    - Not free
**How to Set ?**
    - On your CodeBuild project's configuration.
    - In your CodeBuild buildspec files.
Create Envirnoment Variable in Project Configuration file and use in Build Spec
role for codebuild
```
version: 0.2
env:
  secrets-manager:
    DOCKERHUB_TOKEN: dockerhub:token
    DOCKERHUB_USER: dockerhub:user
phases:
  pre_build:
    commands:
      - echo ${DOCKERHUB_TOKEN} | docker login -u ${DOCKERHUB_USER} --password-stdin
  build:
    commands:
      - docker build -t my-angular-app:latest .
  post_build:
    commands:
      - docker tag my-angular-app:latest ${DOCKERHUB_USER}/my-angular-app:latest
      - docker push ${DOCKERHUB_USER}/my-angular-app:latest
```
## Parameter Srore 
- /docker/hub/token| user
- standard
-Type
    - string
    - securestring
    - stringList
```
version: 0.2
env:
  parameter-store:
    DOCKERHUB_TOKEN: /dockerhub/token
    DOCKERHUB_USER: /dockerhub/user
phases:
  pre_build:
    commands:
      - echo ${DOCKERHUB_TOKEN} | docker login -u ${DOCKERHUB_USER} --password-stdin
  build:
    commands:
      - docker build -t my-angular-app:latest .
  post_build:
    commands:
      - docker tag my-angular-app:latest ${DOCKERHUB_USER}/my-angular-app:latest
      - docker push ${DOCKERHUB_USER}/my-angular-app:latest
```
# Push Images to ECR
```
version: 0.2
env:
  variables:
    ECR_REPO_NAME: my-angular-app
  parameter-store:
    DOCKERHUB_TOKEN: /dockerhub/token
    DOCKERHUB_USER: /dockerhub/user
phases:
  pre_build:
    commands:
      # Docker Hub login
      - echo ${DOCKERHUB_TOKEN} | docker login -u ${DOCKERHUB_USER} --password-stdin 
      
      # ECR login
      - ECR_MAIN_URI="${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
      - aws ecr get-login-password --region ${AWS_REGION} | docker login -u AWS --password-stdin ${ECR_MAIN_URI}

      - ECR_IMAGE_URI="${ECR_MAIN_URI}/${ECR_REPO_NAME}:latest"
  build:
    commands:
      - docker build -t my-angular-app:latest .
  post_build:
    commands:
      - docker tag my-angular-app:latest ${ECR_IMAGE_URI}
      - docker push ${ECR_IMAGE_URI}
```


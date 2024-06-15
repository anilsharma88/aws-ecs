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

# AWS Taks Defination
-  name
- image URL
- container Port
- Envirnoment Variable Apps Envirnoment {FARGETE, EC2}, CPU, Memory, TAKS Role, STORAGE, Managing Logging , revision:1
# ECS Cluster
- name
- VPC
- Subnet
- ec2 autoscaing group
- Monitoring 
# AWS Service
- Capcity provider strategy
- Deployment Configuration
    - service
    - task defination
    - rolling update
    - name
    - LB
# Create a stage deploy use imagedefinitions.json file in deploye of ECS Deploye and upadte build file 
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

      # Generate image definitions file for ECS
      - printf '[{"name":"angular-app","imageUri":"%s"}]' ${ECR_IMAGE_URI} > imagedefinitions.json

artifacts:
  files:
    - imagedefinitions.json
```
- **Use Git Commit ID for Image Tags**


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

      - ECR_IMAGE_URI="${ECR_MAIN_URI}/${ECR_REPO_NAME}:latest { GET it From TAG ID}"
  build:
    commands:
      - docker build -t my-angular-app:latest .
  post_build:
    commands:
      - docker tag my-angular-app:latest ${ECR_IMAGE_URI}
      - docker push ${ECR_IMAGE_URI}

      # Generate image definitions file for ECS
      - printf '[{"name":"angular-app","imageUri":"%s"}]' ${ECR_IMAGE_URI} > imagedefinitions.json

artifacts:
  files:
    - imagedefinitions.json

```
## Use AWS Public Gallery Images to build your docker Image 
```
# syntax=docker/dockerfile:1

# STAGE 1: Build the Angular project
FROM public.ecr.aws/docker/library/node:20 AS builder

# Install Angular CLI
RUN npm install -g @angular/cli@17

# Change my working directory to a custom folder created for the project
WORKDIR /my-project

# Copy everything from the current folder (except the ones in .dockerignore) 
# into my working directory on the image
COPY . .

# Install dependencies and build my Angular project
RUN npm install && ng build -c production


# STAGE 2: Build the final deployable image
FROM public.ecr.aws/docker/library/nginx:1.25

# Allow the HTTP port needed by the Nginx server for connections
EXPOSE 80

# Copy the generated static files from the builder stage
# to the Nginx server's default folder on the image
COPY --from=builder /my-project/dist/my-angular-project /usr/share/nginx/html

------------------------------------
version: 0.2
env:
  variables:
    ECR_REPO_NAME: my-angular-app
phases:
  pre_build:
    commands:
      # ECR Public Gallery login
      - aws ecr-public get-login-password --region us-east-1 | docker login -u AWS --password-stdin public.ecr.aws
      
      # ECR login
      - ECR_MAIN_URI="${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
      - aws ecr get-login-password --region ${AWS_REGION} | docker login -u AWS --password-stdin ${ECR_MAIN_URI}

      - ECR_IMAGE_URI="${ECR_MAIN_URI}/${ECR_REPO_NAME}:${CODEBUILD_RESOLVED_SOURCE_VERSION:0:8}"
  build:
    commands:
      - docker build -t my-angular-app:latest .
  post_build:
    commands:
      - docker tag my-angular-app:latest ${ECR_IMAGE_URI}
      - docker push ${ECR_IMAGE_URI}

      # Generate image definitions file for ECS
      - printf '[{"name":"angular-app","imageUri":"%s"}]' ${ECR_IMAGE_URI} > imagedefinitions.json

artifacts:
  files:
    - imagedefinitions.json
```

#### But if you rolling update failed
 - service try to launch new tasks continually 

### Rollback to rolling Update 
- use **deployment failure detection** in service & rollback
- **EC2 Deployment Circuit Breaker**
  - Threshold: number of failed tasks to container an ECS rolling deployment as a failure
  - Min threshold is 10 & Max threshold is 200.
    - How to calculate circuit brake threshold
      - Desire tasks = 2
      - .5* desired taks = .5*2 = 1
      -  1 <10 
      - Threshold = 10
      - 10 task failure are needed for ECS rolling deployment to be considered as a failure.
- **Deployment circut braker** if the service can't reach a steady state because of task failed to launch, the deployment fails.
- **Min and Max task in rolling Update**
  - Update Service and update min & max for new deploymeny if Min=50 is 50% * desired task  Max=150%* disired task
  - Total taske = Max - min
- **Deployment rollback**
- 
# AWS Capacity Provider 

# AWS Cluster

# ECS services on ASG Capacity providers for Rolling Deployment.
- Create service and define capacity provide ec2 not use default { When configure cluster }
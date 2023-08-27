# Assignment 3.11

A brief description of what tools and procedure used in this assignment.
### Github Repository 

[Procedures of creating new Repository](https://docs.github.com/en/get-started/quickstart/create-a-repo)

### Github Action

[Introduction of Github Action](https://docs.github.com/en/actions/learn-github-actions/understanding-github-actions)

### Visual Studio Code

[Getting started with Visual Studio Code](https://code.visualstudio.com/docs/introvideos/basics)

### AWS ECR

[Getting started with Amazon ECR using the AWS Management Console](https://docs.aws.amazon.com/AmazonECR/latest/userguide/getting-started-console.html)

### AWS ECS

[Set up to use Amazon ECS](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/get-set-up-for-amazon-ecs.html)

[Creating a task definition using the console](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/create-task-definition.html)

#### READme.md

Include the overall steps and tools used in this assignment. 

#### Dockerfile

```bash

FROM public.ecr.aws/bitnami/node:latest

# Create app directory
WORKDIR /usr/src/app

# Install app dependencies
# A wildcard is used to ensure both package.json AND package-lock.json are copied
# where available (npm@5+)
COPY package*.json ./


RUN npm install
# If you are building your code for production
# RUN npm ci --only=production

# Bundle app source
COPY . .

EXPOSE 8080

CMD [ "node", "server.js" ]
  ```

### ecs-task-dfinition.json
```bash

FROM public.ecr.aws/bitnami/node:latest

# Create app directory
WORKDIR /usr/src/app

# Install app dependencies
# A wildcard is used to ensure both package.json AND package-lock.json are copied
# where available (npm@5+)
COPY package*.json ./


RUN npm install
# If you are building your code for production
# RUN npm ci --only=production

# Bundle app source
COPY . .

EXPOSE 8080

CMD [ "node", "server.js" ]
```
### server.js
```bash

FROM public.ecr.aws/bitnami/node:latest

# Create app directory
WORKDIR /usr/src/app

# Install app dependencies
# A wildcard is used to ensure both package.json AND package-lock.json are copied
# where available (npm@5+)
COPY package*.json ./


RUN npm install
# If you are building your code for production
# RUN npm ci --only=production

# Bundle app source
COPY . .

EXPOSE 8080

CMD [ "node", "server.js" ]
```

### .github/workflows/aws.yml
```bash
# This workflow will build and push a new container image to Amazon ECR,
# and then will deploy a new task definition to Amazon ECS, when a release is created
#
# To use this workflow, you will need to complete the following set-up steps:
#
# 1. Create an ECR repository to store your images.
#    For example: `aws ecr create-repository --repository-name my-ecr-repo --region ap-southeast-1`.
#    Replace the value of `ECR_REPOSITORY` in the workflow below with your repository's name.
#    Replace the value of `aws-region` in the workflow below with your repository's region.
#
# 2. Create an ECS task definition, an ECS cluster, and an ECS service.
#    For example, follow the Getting Started guide on the ECS console:
#      https://ap-southeast-1.console.aws.amazon.com/ecs/home?region=ap-southeast-1#/firstRun
#    Replace the values for `service` and `cluster` in the workflow below with your service and cluster names.
#
# 3. Store your ECS task definition as a JSON file in your repository.
#    The format should follow the output of `aws ecs register-task-definition --generate-cli-skeleton`.
#    Replace the value of `task-definition` in the workflow below with your JSON file's name.
#    Replace the value of `container-name` in the workflow below with the name of the container
#    in the `containerDefinitions` section of the task definition.
#
# 4. Store an IAM user access key in GitHub Actions secrets named `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`.
#    See the documentation for each action used below for the recommended IAM policies for this IAM user,
#    and best practices on handling the access key credentials.

on:
  push:
    branches: 
      - main

name: Deploy to Amazon ECS

jobs:
  deploy:
    name: Deploy
    runs-on: ubuntu-latest

    steps:
    - name: Checkout
      uses: actions/checkout@v2

    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v1
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: ap-southeast-1

    - name: Login to Amazon ECR
      id: login-ecr
      uses: aws-actions/amazon-ecr-login@v1

    - name: Build, tag, and push image to Amazon ECR
      id: build-image
      env:
        ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
        ECR_REPOSITORY: yuyuan-3.11
        IMAGE_TAG: latest
      run: |
        # Build a docker container and
        # push it to ECR so that it can
        # be deployed to ECS.
        docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
        docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
        echo "::set-output name=image::$ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG"

    - name: Fill in the new image ID in the Amazon ECS task definition
      id: task-def
      uses: aws-actions/amazon-ecs-render-task-definition@v1
      with:
        task-definition: ecs-task-definition.json
        container-name: task-family
        image: ${{ steps.build-image.outputs.image }}

    - name: Deploy Amazon ECS task definition
      uses: aws-actions/amazon-ecs-deploy-task-definition@v1
      with:
        task-definition: ${{ steps.task-def.outputs.task-definition }}
        service: task-services
        cluster: yuyuan-cluster
        wait-for-service-stability: true
            
```
### Command and Steps to Run in Visual Studio Code under terminal
- Git clone <repository ssh>
- Npm init 
- Npm install serverless - package.json will appear
- Create server.js
- Npm install serverless-offline- package-lock.json will appear
- Move the node_module into .gitignore
- Create Dockerfile.
- Create ECS and ECR in Aws console.
- Create task definition in Aws console under ECR and include the task name into the tak-definition.json
- Create cluter under ECS and include the cluster name into aws.yml file. 
- Create aws.yml under file .github/workflows, input login to ECR, create docker image and push into ECS inside the aws.yml.
- Carry out Github Action and Run the workflow.

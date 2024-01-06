## AWS ECS With Self managed 
Amazon Elastic Container Service is a managed container orchestration service which allows you to deploy and scale containerized applications. An overview of the features and pricing can be found at the [AWS website](https://aws.amazon.com/ecs).

ECS consists out of a few components:

- **`Elastic Container Repository (ECR)`**: A Docker repository to store your Docker images (similar as DockerHub but now provisioned by AWS).
- **`Task Definition`**: A versioned template of a task which you would like to run. Here you will specify the Docker image to be used, memory, CPU, etc. for your container.
`Tasks in ECS` are like jobs that tell the computer what to do. For example, you could have a task that says "run this specific program in a container". 
`ChapGPT analogy of ECS Task` is a chef cooking a meal in a restaurant kitchen.
- **`ECS Cluster`**: The Cluster definition itself where you will specify how many instances you would like to have and how it should scale.
- **`Service`**: Based on a Task Definition, you will deploy the container by means of a Service into your Cluster. `Services in ECS` make sure there's always a container running your program, so you don't have to keep starting it yourself. `ChapGPT analogy of ECS Service` is like ensuring that there is always a steady supply of that meal available to customers, even during peak hours.

At the core of it, below are the key steps involved:-

- You will create a Docker image for your application (for example Spring Boot Application, Python App, Python Flask Web App, Node App etc.)
- We will upload the image to ECR
- We will create a Task Definition for the image
- We will create a Cluster
- We will create the service and deploy the container by means of a Service to the Cluster.
- It is highly recommended to front the application running in ECS with an Application Load Balancer.

## ECS Task Logs

Below are the configs of a terraform script as I was using Terraform for provisioning my ECS, I had to add the following configuration to my `container_definitions` inside `ecs_task_definition`.

```
logConfiguration= {
  logDriver= "awslogs",
  options= {
      awslogs-create-group= "true",
      awslogs-group= "callums-node-logs",
      awslogs-region= "eu-west-3",
      awslogs-stream-prefix= "awslogs-example"
  }
},
```

By specifying the 'awslogs' driver in the log configuration, we enable the visibility of logs generated from within the container. I had to specify things such as aws-create-group to create the log group if it didn’t already exist and explicitly give the log group a name. There are several other options for [configuring AWS logs](https://docs.aws.amazon.com/AmazonECS/latest/APIReference/API_LogConfiguration.html).

However, as always in AWS, just enabling the feature isn’t enough. By default, your IAM role won’t have the required permissions. Ensure that the IAM role for your Amazon ECS container instance has **`logs:CreateLogStream`** and **`logs:PutLogEvents`** permissions. Since I was using Fargate, I also had to give these roles to my ECS task execution IAM role as well.
# System Design for Model Deployment to Production

Let's take a look here how can be designed an architecture<br>
serving machine learning model interference via web application.

There is only a few things so rewarding<br>
as seeing your work in production serving to others. <br>
Be it 5, 100, or million users.<br>
<br>
To scale from zero to millions, <br>we need to think about the overall system design<br> that can look like on the picture below:

![Model Deployment Architecture](model_deployment_architecture.png)
- This architecture is designed to serve users 24/7,<br>
providing the interface in real-time,<br>
and scale automatically with the load.
- The web server is optimized to provide immediate response to users<br>
while minimizing maintenance & costs for infrequent access with AWS Fargate.
- The network response time is optimized for customers located in Switzerland 
while covering the whole Western Europe with 23ms to reach distant Ireland.
- The interference pipeline supports several options from traditional computing services, to AWS SageMaker, to my favourite KubeFlow.

Now, what are the main steps to get design for your use-case?

## 1. Microservices decomposition
In the first step, we can decompose the whole setup into microservices. <br>
That offers us several advantages: 
1. Better scalability where each component can be scaled independently based on its load.<br>
Additionally, each microservice can run on its own optimal harware. <br>
Imagine that your interference pipeline requires GPU. <br> You don't want to pay for expensive GPU to run your web app.
2. Both parts can be easily entrusted to different people or teams <br>
and handled independently including their CI/CD pipelines.

For this example we can split it into:
- I. Dockerized and application web-server
- II. Dockerized and interference pipeline exposed via REST API

## 2. Scaling-service selection
In the second step, we can focus on how we run and scale each microservice.

I will use here AWS services,<br>
but the though process is the same for any other cloud provider.<br>
If you use Microsoft Azure or Google Cloud Platform<br>
you can just replace the services here for twins of your provider.

### 2.1. Scaling-service for Application Webserver
For webserver deployment we have the following options:
- a) Kubernetes spinning traditionall servers (AWS ECS: EC2)
    - Ideal when we face stable load and need to keep control about underlaying hardware <br>
    Cons: Maintaince of servers, updates, etc.<br>
    Pros: Flexibility, the most price effective for constant load
- b) Serverless (AWS Lambda)
    - Ideal when the service is not accessed most of the time<br> 
    and user can tollerate response delay due to cold starts of lambda container<br>
    Cons: Cold starts that makes your user wait  before lambda starts your container<br>
    Pros: No maintaince of servers or images, you just provide the code.<br>
    You are charged for number of times the function is called. Not for idle CPU time.
- c) Kubernetes spinning containers (AWS ECS: AWS Fargate) 
    - Ideal when we want immediate response to users<br>
    while don't want to spend time maintaining the servers<br>
    Cons: Maintaince of containers in contrast with serverless lambda<br>
    Pros: No need to maintain servers in consrast with EC2.<br> 
    You pay for CPU time you use. Not for idle CPU time as with EC2. 

For this case I went with using AWS Fargate (c).

### 2.2. Scaling-service for Model deployment
Now things get interesting.<br>
In addition to scaling and monitoring service health,<br>
you want to monitor ML-specific things<br>
like model accuracy or data drifts<br>
so you know when is time to retrain your model. 

You can use general computing services mentioned above<br>,
and take care about logging ML-specific things on your model level<br>
using built-in Cloud Watch or open-source Prometheus with Grafana.

Or you can use tooling specialized for model deployment
that has this monitoring by default.

Your choice is between<br>
cloud-native solution which is Sagemaker for AWS<br>
or open-source solution you integrate with ECS when my first choice would be Kubeflow.

## 3. Database selection
When you scale from zero to million users<br>
it's the database scaling that causes worries.

Your choice is between traditional relational databases like PostgreSQL<br>
or noSQL solutions like MongoDB.

For this case it was about handling relational data<br>
making it easy for me to stick with relational database.<br> 
However, it would work the same with noSQL<br>
that would offer advantage of horrizontal scaling.

## 4. Model deployment

The beauty of this microservices architecture<br> 
is that the model interference pipeline is a black box<br> 
where we just call defined entry point serving as model load balancer.

The deployment after the model retraining is matter of CI/CD pipeline.<br> 
With blue-green deployment, the CI/CD pipeline deploys new instances with the new version<br> 
and redirect the traffic to this new version.

## Summary
We went through possible system design<br>
serving machine learning model interference via web application<br>
and necessary engineering behind to scale from zero to million of users.

![Model Deployment Architecture](model_deployment_architecture.png)

Now depending on the intended scale<br> 
several more technique could be applied<br> 
as horizontal scaling of read replicas, caching, <br>
or using message queues to provide buffer for sudden spikes.

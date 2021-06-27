# ML Model Deployment to Production

Below sharing a sample architecture I did in the past.

- The architecture is designed to serve users 24/7,<br>
providing the interface in real-time,<br>
and scaling automatically with the load to serve up to thousands of requests per day.
- The network response time is optimized for customers located in Switzerland<br> 
while covering the whole Western Europe with 23ms to reach distant Ireland. <br>
Integration with USA is possible with 147 ms to reach California.
- The web server is optimized to provide immediate response to users<br>
while minimizing maintenance & costs for infrequent access with AWS Fargate.
- The interference pipeline supports several options from AWS Lambda<sup>[1]</sup>, AWS SageMaker, to my favourite KubeFlow.

![Model Deployment Architecture](model_deployment_architecture.png)

[1] Using AWS Lambda for interference is possible only when run time is below 300s with maximum memory of 3048 MB.
# App Deployment Steps

## Frontend
1. Register a domain using a Domain Registrar
2. Create an S3 bucket with the same name as the root domain and add frontend files
3. Use Route 53 as the DNS service for the domain 
   1. Create a hosted zone (name hosted zone the root domain)
   2. Add Route 53 nameservers for your hosted zone to the Domain Registrar used to register domain
   3. Make sure nameservers have propagated (Can take up to 48 hours)
4. Create a CloudFront distribution with the origin as S3 bucket (will add bucket policy onto root domain S3 bucket)
5. Add root domain and subdomain as alternate domains for CF distribution
6. Request SSL Certificate from AWS Certificate Manager
   1. Include root domain and (www) subdomain in certificate
   2. ACM will add CNAME records and A/AAAA records to your hosted zone. If not, they will need to be done manually.
   3. Once certificate is issued, add certificate to CF distribution
   5. Add index.html as the default root object in the CF distribution
   6. Wait for CF ditribution to redeploy
  
## Database
1. Go to AWS RDS to create a database
   - Choose full configuration
   - Free tier
   - Create username and password
   - Choose instance configuration (Burstable classes / db.t3.micro)
   - Storage type: General Purpose SSD, 20 GiB
   - Choose VPC (must be the same that ECS tasks use)
   - Create and name new VPC security group
   - Port: 5432
2. Add the database endpoint to Flask app
3. Create environment variable in AWS Secrets Manager console
   - Add a secret for the database URL
   - Choose other type of secret
   - Add database URL in plain text

## Backend
1. Create an IAM user to handle ECR and ECS
   1. Attach Policies:
      - AmazonEC2ContainerRegistryFullAccess - allows user to use AWS ECR
      - AmazonECS_FullAccess - allows user to use AWS ECS
   2. Generate access key for the user
3. Create repository for ECS image in AWS ECR
4. Create an ECS Cluster
5. Create a Task Definition
   - Name the task definition family
   - Launch type = Fargate
   - CPU = .5 vCPU
   - Memory 2 GB
   - Create default task execution role
   - Container
     - Name container
     - Add image URI (add :latest at the end)
     - Define port mapping = TCP Port 80 HTTP (Gunicorn is listening on port 80 inside the container)
6. Revise the task definition with JSON to include the secret made using the secret arn
7. Create and attach an inline policy to the task execution role to read secrets from AWS Secrets Manager
8. Create ALB in EC2 console
   - Add listener for HTTP and HTTPS
     - Request new ACM certificate for api.banksie.app
     - add a record to Route 53 for api.banksie.app (will need time to propagate)
   - create security group for ALB
     - allow inbound HTTP and HTTPS traffic from anywhere
     - allow all outbound traffic to ECS tasks (the banksie-sg)
   - create target group for ecs tasks (For an IP but do not add any targets)
     - Add health check path as /health
     - in Flask app add route for /health to return code 200
   - Add ALB url to frontend code files
   - Add new files to S3 bucket
9. Create a service
   - Add task definition previously created
   - Choose capacity provider strategy (FARGATE)
   - Desired tasks = 1
   - Networking
     - Choose VPC (same as database)
     - Choose at least 2 subnets
   - Create security group that allows HTTP/HTTPS traffic from ALB on port 80
   - Add ALB that was previously created
   - Service will have an error until we push an image
10. In EC2 console, go to security groups
   - Choose the RDS security group made previously
   - Add inbound rule so the task security group can access the RDS
11. Push Docker image to ECR via AWS CLI
   1. Configure AWS CLI credentials
   2. Log into ECR 
   3. Build/tag docker image
   4. Push image
12. Force new deployment for your service with the new image and revised task to include the image
13. Set up Github Actions for automatic deployments
    1. Add secrets and variables to the Github repository
    2. Create workflow (.yml file)

## Services Used
* Amazon Route 53 ($0.50/m)
* Amazon Cloud Front (First 1TB free/m)
* Amazon S3 Free Tier
* Amazon ECS
* Amazon ECR
* Amazon IAM
* Github Actions
* CMD
* Docker
* AWS RDS
* AWS EC2 (Security Groups and Load Balancer)
* AWS Secrets Manager
* AWS CloudWatch

## What I Learned
- How to use Route 53 DNS service for an existing domain
- How to create alias records in Route 53 to access AWS resources
- How to create an SSL certificate with AWS Certificate Manager and attach it to a CloudFront Distribution/ALB
- How to create cache invalidations
- AWS ECR and how to push an image via the CLI
- ECS service deployment rollbacks use the image digest saved in a container and not the latest image created
- How to add secrets and variables to Github repository
- How to automate deployment with Github Actions
- How to set up ALB health checks for an app
- How to use CloudWatch to debug my ECS tasks
- How to set up secure security groups
- How to create secrets in AWS Secrets Manager

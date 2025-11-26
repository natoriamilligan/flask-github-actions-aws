# App Deployment Steps

## Frontend
1. Register a domain using a Domain Registrar
2. Create an S3 bucket with the same name as the root domain
3. Use Route 53 as the DNS service for the domain 
   1. Create a hosted zone (name hosted zone the root domain)
   2. Add Route 53 nameservers for your hosted zone to the Domain Registrar used to register domain
   3. Make sure nameservers have propogated. (Can take up to 48 hours)
4. Create a CloudFront distribution with the origin as S3 bucket (will add bucket policy onto root domain S3 bucket)
5. Add root domain and subdomain as alternate domains for CF distribution
6. Request SSL Certificate from AWS Certificate Manager
   1. Include root domain and (www) subdomain in certificate
   2. Add CNAMES to hosted zone in Route 53 (ACM did this automatically and added A and AAAA records to hosted zones. or do manually)
   3. Once certificate is issued, add certificate to CF distribution
   5. add index.html as the default root object
   6. Wait for CF ditribution to redeploy
7. Add frontend files to S3 bucket

## Backend
1. Create IAM user to handle ECR and ECS
   1. Attach Policies:
      - AmazonEC2ContainerRegistryFullAccess - allows user to use AWS ECR
      - AmazonECS_FullAccess - allows user to use AWS ECS
   2. Generate access key
3. Create repository for ECS image
4. Create ECS Cluster
5. Create a Task Definition
   - Name the task definition family
   - Launch type = Fargate
   - CPU = .5 vCPU
   - Memory 2 GB
   - Create default task execution role
   - Container
     - Name container
     - Add image URI
     - Define port mapping = TCP Port 5000 HTTP
6. Create a service
   - Add task definition previously created
   - Choose capacity provider strategy (FARGATE)
   - Desired tasks = 1
   - Networking
     - Choose default VPC
     - Choose at least 2 subnets
   - Service will have an error until we push an image (Next Step)
7. Push Docker image to ECR via AWs CLI
   1. Configure AWS CLI credentials
   2. Log into ECR (add command here and wherever else needed)
   3. Build docker image
   4. Tag image
   5. Push image
8. Force new deployment for your service 
9. Confirm that image was successfully push to the ECR repository via the public IP
10. Set up Github Actions for automatic deployments
    1. Add secrets and variables to the Github repository
    2. Create workflow and push to Github

## Database
1. Go to AWs RDS to create a database
   - Choose full configuration
   - Free tier
   - Create username and password
   - Choose instance configuration (Burstable classes / db.t3.micro)
   - Storage type: General Purpose SSD, 20 GiB
   - Choose same VPC as ECS task
   - Create a new VPC security group
   - Port: 5432
2. In EC2 console, go to security groups
   - Choose the RDS security group just made
   - Add inbound rule so the task security group can access the RDS
3. Add database url to local app




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
* AWS EC2 (Security Groups)
* AWS Secrets Manager

## What I Learned
- How to use Route 53 DNS service for an existing domain
- How to create alias records in Route 53 to access AWS resources
- How to create an SSL certificate with AWS Certificate Manager and attach it to a CloudFront Distribution
- How to create cache invalidations
- AWS ECR and how to push an image via the CLI
- ECS service deployment rollbacks use the image digest saved in a container and not the latest image created
- How to add secrets and variables to Github repository
- How to automate deployment with Github Actions

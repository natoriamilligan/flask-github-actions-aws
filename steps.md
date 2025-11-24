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
1. Create IAM user to handle Github Actions
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
8. Set up Github Actions for automatic deployments





## Services Used
* Amazon Route 53 ($0.50/m)
* Amazon Cloud Front (First 1TB free/m)
* Amazon S3 Free Tier

* Amazon ECS
* Amazon ECR

## What I Learned
- How to use Route 53 DNS service for an existing domain
- How to create alias records in Route 53 to access AWS resources
- How to create an SSL certificate with AWS Certificate Manager and attach it to a CloudFront Distribution
- How to create cache invalidations

# Banksie App Deployment

## ❓ Overview
In this project, I successfully deployed a multi-tier cloud-based banking app in AWS using S3, ECS, and RDS. 

## 🏗️ Architecture Setup

### 🖥️ Frontend
   1. Registered a domain using a domain registrar (banksie.app)
   2. Created a private S3 bucket with the same name as the root domain and added frontend files
   3. Used Route 53 as the DNS service for the domain 
      - Created a hosted zone (named hosted zone the root domain: banksie.app)
      - Added Route 53 nameservers for the hosted zone to the domain registrar used to register domain
      - Made sure nameservers propagated (Can take up to 48 hours)
   4. Created a CloudFront distribution with the origin as the S3 bucket 
   5. Added root domain (banksie.app) and subdomain (www.banksie.app) as alternate domains for CF distribution
   6. Requested SSL Certificate from AWS Certificate Manager
      - Included root domain and subdomain in certificate
      - ACM added CNAME records and A/AAAA records to the hosted zone, but this can be done manually
      - Once certificate was issued, added certificate to CF distribution
      - Added index.html as the default root object in the CF distribution
      - Wait for CF distribution to redeploy
     
### 🛢️ Database
   1. Create a database in RDS
      - Choose full configuration
      - PostgreSQL
      - Free tier
      - Created username and password
      - Choose instance configuration (Burstable classes / db.t3.micro)
      - Storage type: General Purpose SSD, 20 GiB
      - Choose VPC (must be the same that ECS tasks use)
      - Created new VPC security group to be configured later
      - Port: 5432
   2. Added the database endpoint to Flask app
   3. Created environment variable in AWS Secrets Manager console for database URL
      - Added a secret for the database URL
      - Choose "other type of secret"
      - Added database URL in plain text
     
### ⚙️ Backend


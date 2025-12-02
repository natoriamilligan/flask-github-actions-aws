# Banksie App Deployment

## ❓ Overview
In this project, I successfully deployed a multi-tier cloud-based banking app in AWS using S3, ECS, and RDS. 

## 🏗️ Architecture Setup

### 🖥️ Frontend
1. Registered a domain using a domain registrar (banksie.app)
2. Created a private S3 bucket with the same name as the root domain and added frontend files
3. Used Route 53 as the DNS service for the domain 
   1. Created a hosted zone (named hosted zone the root domain: banksie.app)
   2. Added Route 53 nameservers for the hosted zone to the domain registrar used to register domain
   4. Made sure nameservers propagated (Can take up to 48 hours)
4. Created a CloudFront distribution with the origin as the S3 bucket 
5. Added root domain (banksie.app) and subdomain (www.banksie.app) as alternate domains for CF distribution
6. Requested SSL Certificate from AWS Certificate Manager
   1. Included root domain and subdomain in certificate
   2. ACM added CNAME records and A/AAAA records to the hosted zone, but this can be done manually
   3. Once certificate was issued, added certificate to CF distribution
   5. Added index.html as the default root object in the CF distribution
   6. Wait for CF distribution to redeploy

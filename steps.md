# App Deployment Steps
1. Register a domain using a Domain Registrar
2. Create 2 S3 buckets with the same name as the domain (1 with the root domain and 1 with the subdomain)
3. Enable website hosting for the subdomain and redirect to the root domain bucket
4. Create a CloudFront distribution with the origin as the root domain S3 bucket (will add bucket policy onto root domain S3 bucket)
5. Use Route 53 as the DNS service for the domain 
   1. Create a hosted zone
   2. Add Route 53 nameservers for your hosted zone to the Domain Registrar used to register domain
   3. Create an A record for the root domain pointing to the CloudFront distribution
      - Click Alias -> banksie.app (CF distribution)
   4. Create an A record for the subdomain S3 bucket
6. Make sure nameservers have propogated. (Can take up to 48 hours)





## Services Used
* Amazon Route 53 ($0.50/m)
* Amazon Cloud Front (First 1TB free/m --)


## What I Learned
- How to use Route 53 DNS service for an existing domain
- How to create alias records in Route 53 to access AWS resources
- How to redirect an S3 bucket to another S3 bucket

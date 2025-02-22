# Comprehensive Guide: Setting Up a Static Website on AWS

## A Complete Process for Beginners

## Overview

This guide walks you through the complete process of setting up a static website on AWS, from domain registration to final deployment. Each step includes detailed explanations of why certain choices are made and how different AWS services work together.

First create AWS account, free tier if you want a basic account where you can host your website.

## Step 1: Domain Registration and DNS Setup

### Register Your Domain

1. Navigate to Route 53:

   ```
   AWS Console → Search "Route 53" → Click "Route 53"
   ```

   **Why Route 53?** Using Route 53 for domain registration simplifies the process as it integrates directly with other AWS services, eliminating the need to manually update nameservers or transfer domains.

2. Register Domain:

   ```
   Route 53 → Domains (find it in left sidebar) → Registered domains → Register domain
   ```

   **Configuration Steps:**

   - Enter desired domain name (e.g., yourdomain.com)
   - Check availability
   - Complete contact details
   - Choose privacy protection (recommended for personal sites)
   - Review and complete purchase
   - Wait for registration confirmation email

   **Important Notes:**

   - Registration takes up to 24 hours or longer but for new accounts consider emailing aws support to expedite
   - Status will show as "pending"
   - You'll receive email when complete
   - Keep contact information accurate for domain ownership
   - Privacy protection hides personal info from WHOIS lookups
   - Save confirmation email for records
   - Keep track of domain expiration date

## Step 2: SSL Certificate Setup

Why do this early? Certificate validation can take time, and you'll need it ready for CloudFront.

1. Request Certificate:

   ```
   Switch to us-east-1 region or your exact region → Certificate Manager → Request certificate
   ```

   **Critical Region Note:** Certificate MUST be in your region for CloudFront use.

2. Certificate Configuration:

   ```
   Request public certificate:
   - Add domain: yourdomain.com
   - Add domain again and input wildcard starting with asterisk: *.yourdomain.com   (Alternatively just provide the www.yourdomain.com)
   - Choose DNS validation (recommended) instead of email validation.
   ```

   ### Expected Outcome

   The certificate will enter a "Pending validation" state. We'll complete the validation in a later task using Route 53

   **Why These Choices?**

   - Public certificate: Required for public websites
   - Including wildcard: Covers all subdomains
   - DNS validation: Automatic renewal, no manual intervention

   ### Why SSL Matters

   Security is crucial for modern websites. An SSL certificate enables HTTPS, providing encrypted connections between your website and its visitors.

## Step 3: S3 Bucket Creation and Configuration

### Create Bucket

1. Navigate to S3:

   ```
   AWS Console → Search "S3" → Create bucket
   ```

2. Bucket Configuration:
   ```
   Name: yourdomain.com (exactly match domain)
   Region: Choose based on target audience
   Unblock public access (required for website)
   ```
   **Why These Settings?**
   - Matching domain name: Consistency and clarity
   - Region choice: Affects latency and costs
   - Public access: Required for website hosting

### Why S3 for Static Websites?

Amazon S3 provides reliable, scalable storage perfect for hosting static website files. It's cost-effective and integrates well with CloudFront for content delivery.

### Enable Static Website Hosting

1. Basic Setup:

   ```
   Click on your newly created bucket → Properties →  Scroll to bottom to find Static website hosting → Select Enable
   - Enter "index.html" for Index document (First create one if you haven't already)
   - Error document: error.html (optional)
   ```

   **Why This Configuration?**

   - Index document: Default page for directories
   - Error document: Custom error handling
   - Enables proper web serving behavior

2. Bucket Policy:
   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Sid": "PublicReadGetObject",
         "Effect": "Allow",
         "Principal": "*",
         "Action": "s3:GetObject",
         "Resource": "arn:aws:s3:::your-bucket-name/*"
       }
     ]
   }
   ```
   **Policy Explanation:**
   - Allows public read access
   - Restricts to GetObject only
   - Applies to all files in bucket
   - Minimum required permissions

### Expected Outcome

Your bucket will have a website endpoint URL, but don't use this as your final website URL – we'll set up CloudFront for that.

## Step 4: Route 53 DNS Configuration

### Why DNS Configuration Matters

DNS connects your domain name to your website's actual location. Proper DNS configuration ensures visitors can find your website using your domain name.

### Create/Update Records

1. Navigate to Hosted Zone:

   ```
   Route 53 → Hosted zones → Your domain
   ```

   If registered through Route 53, hosted zone is created automatically.

   If not then create it by following the steps below

   ### Create Hosted Zone

2. Navigate to Route 53:

   ```
   - Search for "Route 53"
   - Click "Hosted zones"
   - Click "Create hosted zone"
   ```

3. Configure Zone:

   ```
   - Enter your domain name
   - Select "Public hosted zone"
   - Click "Create hosted zone"
   ```

4. Certificate Validation:
   ```
   Create validation CNAME records:
   - Use "Create records in Route 53" button in ACM
   - Wait for certificate status "Issued"
   ```
   **Why Important?**
   - Proves domain ownership and completes the SSL certificate validation.
   - Required for HTTPS
   - Automatic with Route 53

## Step 5: CloudFront Distribution Setup

### Create Distribution

1. Navigate to CloudFront:

   ```
   AWS Console → CloudFront → Create distribution
   ```

2. Origin Settings:

   ```
   Origin domain: yourdomain.com.s3-website-region.amazonaws.com
   Important: Use S3 website endpoint, not bucket endpoint. This is crucial because the website endpoint handles proper HTTP behavior and index document routing.
   ```

   - Protocol: HTTP only
   - Name: Leave as default

   **Why Website Endpoint?**

   - Supports proper HTTP behavior
   - Handles redirects correctly
   - Works with index documents

3. Cache Behavior:

   ```
   - Redirect HTTP to HTTPS
   - Compress objects automatically
   - Cache policy: CachingOptimized
   - Allowed methods: GET, HEAD
   ```

   **Reasoning:**

   - HTTPS: Security standard
   - Compression: Better performance
   - Cache policy: Optimized for static content
   - Limited methods: Static site needs

4. Distribution Settings:

   ```
   - Alternate domain names: yourdomain.com, www.yourdomain.com
   - Custom SSL certificate: Select your certificate
   - Default root object: index.html
   ```

   **Configuration Benefits:**

   - Supports both www and root domain
   - Secure HTTPS connection
   - Proper index page handling

     -Also choose your preferred price class

## Step 6: Final DNS Configuration

## Why Final DNS Updates Matter

The last step connects everything together, directing your domain name to your CloudFront distribution.

### Update Route 53 Records

1.  Create A Record:

```
- Go to Route 53 Hosted Zones
- Select your domain
- Click "Create record"
- Leave name empty for root domain
- Select "A" record
- Toggle "Alias"
- Select CloudFront distribution
- Click "Create records"
```

2. Create WWW Record:

   ```
   - Click "Create record"
   - Enter "www" for name
   - Select "A" record
   - Toggle "Alias"
   - Select same CloudFront distribution
   - Click "Create records"
   ```

   ```
   ***Why This Setup?***
   ```

- A records for apex domain
- Alias for AWS integration
- Covers both www and root

## Step 7: Testing and Verification

### Why Testing is Critical

Thorough testing ensures all components are working together correctly and your website is accessible to visitors.

### Complete Testing Checklist

1. Check Website Access:

   ```
   Test all access points:
   - https://yourdomain.com
   - https://www.yourdomain.com
   - HTTP to HTTPS redirect
   ```

2. Verify Content Delivery On Cloufront:

   ```
   - Check distribution status (Deployed)
   - Verify HTTPS redirect
   - Test page loading and caching

   Verify:
   - Page loads correctly
   - SSL certificate works
   - Files are cached
   - create invalidations in cloudfront using /index.html to refresh dns cache if webpage not updating
   - Verify CloudFront caching is working by accessing the site multiple times.
   ```

### Expected Outcome

Your website should be accessible securely through your domain name, with proper redirects and good performance.

## Important Timelines and Waiting Periods

### Process Timeline

```
1. Domain Registration: Up to 24 hours
2. Certificate Validation: 2-24 hours
3. DNS Propagation: 24-48 hours
4. CloudFront Distribution: 15-20 minutes
```

**Recommendation:** Start with domain registration and certificate request while working on other components.

## Cost Breakdown

```
1. Domain Registration: Annual fee varies by TLD
2. Route 53: ~$0.50/month per hosted zone
3. S3: Minimal for static sites
4. CloudFront: Pay for data transfer
5. ACM: Free with CloudFront
```

## Maintenance Guidelines

1. Regular Tasks:

   ```
   - Monitor certificate expiration
   - Update content via S3
   - Check CloudFront metrics
   - Check cost explorer to see how much you are spending based on utilized AWS resources
   - Review security settings
   - Keep track of domain registration renewal
   ```

2. Future Considerations:
   ```
   - Content backup strategy
   - Traffic monitoring setup
   - Performance optimization
   - Security updates
   ```

## Troubleshooting Tips

1. Common Issues:

   ```
   - DNS not resolving: Check nameservers
   - HTTPS not working: Verify certificates issued
   - Content not updating: Check CloudFront cache, create invalidation for index by /index.html can also do /* for all files in s3 bucket
   - Incorrect Hosted Zone Name Servers
   - CNAME Certificate pending, make sure when creating a record for it in route 53 that it all values are the same. Alternatively create record in route 53 directly through acm where you create the certificate. Can delete and create new one if it times out from pending too long and try it this way
   ```

2. Resolution Steps:

   ```
   - Use Route 53 test record feature, or email amazon support for the correct name servers of your hosted zone
   - Verify CloudFront distribution status
   - Check S3 bucket permissions

   ```

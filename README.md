# Static-Website-Hosting-with-AWS-Services

# Overview

A static website serves prebuilt, fixed content such as HTML, CSS, JavaScript, and media files without executing business logic or querying a database. An example is GitHub Pages.

This project demonstrates how **Amazon S3**, **Amazon CloudFront**, **Amazon Route 53**, and **AWS Certificate Manager (ACM)** integrate to host a fast, scalable, secure, and highly available static website with low latency global content delivery.

S3 stores the static website files and serves as the origin for the CloudFront distribution. CloudFront improves performance by caching content at edge locations worldwide and delivering it from the nearest location to users. Route 53 manages DNS resolution and routes user requests for the custom domain to the CloudFront distribution. ACM provisions and manages the SSL/TLS certificate used by CloudFront, enabling HTTPS and secure communication between clients (browsers) and the distribution.

# Architecture Diagram

<img width="565" height="214" alt="diagram architecture" src="https://github.com/user-attachments/assets/1ac86fbb-d535-467a-b38e-08b8088c1f9b" />


## Request Flow
- **1.** Users access the static website using the custom domain from anywhere in the world
- **2.** The request first hits Route 53, which performs DNS resolution by mapping the custom domain to the associated CloudFront distribution (since CloudFront is the endpoint for the custom domain)
- **3.** The client (user's browser) establishes a secure HTTPS connection using the SSL/TLS certificate provided by Certificate Manager, and sends the request to the CloudFront distribution
- **4.** CloudFront receives the request at the nearest edge location. If the requested content is already cached (cache hit), CloudFront serves it immediately from the edge location, and if the content is not cached (cache miss), CloudFront retrieves the content from the configured origin (S3 bucket) using Origin Access Control (OAC) which prevents direct public access to the bucket
- **5.** Once CloudFront retrieves the content, it caches it at the edge location for future requests and delivers the content to the user
- **6.** This process reduces latency by serving content closer to users, minimizes direct requests to the origin S3 bucket, improves performance, and enables secure, efficient global content delivery

Edge locations are globally distributed AWS data centers that cache content and act as Points of Presence (PoPs) for AWS services enabling data to be served from the location closest to the end user which significantly reduces latency and the distance data must travel

## Benefits
- **Low latency, high availability, and redundancy**: CloudFront delivers content from edge locations close to users, reducing latency and improving load times. It also improves fault tolerance and availability by routing trafffic automatically to the next closest edge location to a user if an edge location becomes unavailable
- **High durability**: S3 provides high durable object storage designed to protect against data loss. It can store unlimited data and serve static website content reliably over the internet
- **Automatic scalability**: S3, CloudFront, and Route 53 are managed services by AWS that automatically scale to handle traffic demands without manual intervention
- **Improved security**: Certificate Manager provisions and manages the SSL/TLS certificate attached to the CloudFront distribution, enabling HTTPS and encrypting data in transit. Origin Access Control (OAC) ensures only CloudFront can access the private S3 bucket
- **Cost efficiency and simplicity**: this architecture reduces operational costs and complexity by eliminating server provisioning and management and, also uses a pay-as-you-go pricing model to optimize costs


## Architecture Components
- **Amazon S3**: static website storage and origin for CloudFront
- **Amazon CloudFront**: global content delivery and edge caching
- **Amazon Route 53**: DNS resolution and domain routing
- **AWS Certificate Manager (ACM)**: SSL/TLS certificate provisioning and management


## Prerequisites
- An active AWS account
- A registered custom domain name (Route 53 can be used as the domain registrar, or another provider such as GoDaddy or IONOS)


# Deployment Steps (with Screenshots)

  ## 1. Create S3 bucket
- In AWS console, search and open S3
- Choose **Create bucket**
- Select **General purpose** as bucket type
- Enter a unique bucket name in **Bucket name** field
- Keep other settings as default
- Ensure **Block all public access** is enabled (this restricts direct public access to the bucket to enable access only via CloudFront)
- Enable **Bucket Versioning** and **Create bucket** 


<img width="1816" height="646" alt="s3" src="https://github.com/user-attachments/assets/4d91ef89-122f-4db8-a822-480237ac3ec0" />

 
 ## Upload Website Content
- Open the created bucket
- Go to the **Objects** tab and choose **Upload**
- Select **Add files** to upload the website HTML file and choose **Upload**


<img width="1794" height="595" alt="file upload" src="https://github.com/user-attachments/assets/a2bf4d64-fc10-4e19-8bfd-77b4ee0fb59d" />


## 2. Create a CloudFront Distribution and Configure Bucket Policy
- In AWS console, search and open CloudFront
- Select **Create distribution**
- Choose **Free** plan and choose **next**
- Enter a name for the distribution 
- Select **Amazon S3** as **Origin type**
- Choose **Browse S3** and select S3 bucket 
- Leave the other settings as default, review settings and choose **Create distribution**
- Select the distribution, under the **General** tab choose **edit**
- In **Default root object - _optional_** field, enter the name of the HTML file and select **Save changes** (this returns to the CloudFront homepage)
- Select the distribution and choose **Edit** 
- Ensure **Origin access control settings (recommended)** is selected
- In **Origin access control** section, select **Copy policy**
- On S3 bucket page, under **Permissions** tab, locate **Bucket policy** section and choose **Edit**
- Delete the default policy, paste the policy copied from CloudFront page, and choose **Save changes** (this creates S3 bucket policy for CloudFront to permit it to fetch content)
- Return to CloudFront page, select **Distributions** under domain tab, copy the distribution domain name and paste on a new browser tab (this displays the content of the website served through CloudFront)

<img width="1836" height="745" alt="cloudfront setting" src="https://github.com/user-attachments/assets/34932755-8fad-40c3-a902-fde4106ac969" />



<img width="1912" height="950" alt="static cloudfront" src="https://github.com/user-attachments/assets/0a89a070-8bc6-41ca-a42b-506df257b72c" />


 ## 3. Create Hosted Zone in AWS Route 53
- In AWS console, search and open Route 53
- Choose **Create hosted zone**
- Enter the custom domain name in **Domain name** field
- Leave other settings as default and choose **Create hosted zone**
- Under **Records** tab, locate and copy **Nameservers (NS)** records (they are four in total)
- Go to the domain registrar (in my case, GoDaddy.com) and replace the default nameservers with the ones from Route 53. This will enables Route 53 to manage traffic for the custom domain

<img width="1894" height="892" alt="godady dns" src="https://github.com/user-attachments/assets/67198f83-82b5-4045-b482-81b6046faf97" />



 ## 4. Generate SSL Certificate For CloudFront Distribution Using ACM
 - In AWS console, search and open Certificate Manager
 - Choose **Request certificate** 
 - Leave **Request a public certificate** as selected, choose **Next**
 - Enter the custom domain name in **Fully qualified domain name** field
 - Ensure **DNS validation - recommended** is selected, leave other settings as default, and choose **Request**
 - (**Ensure the certificate is created in the us-east-1 (N.Virginia) region as CloudFront only supports certificates from this region**)
 - After requesting, the certificate status will show as **Pending validation**. Wait for DNS to propagate and refresh until the status changes to **Issued**
   
<img width="1816" height="502" alt="acm" src="https://github.com/user-attachments/assets/f33327c6-4779-4c55-b90c-ff8ab7e34077" />



<img width="1243" height="577" alt="acm certificate" src="https://github.com/user-attachments/assets/97165399-27a8-4131-90cd-e76b010eb038" />



## 5. Create DNS Record and Attach SSL to CloudFront
- In Route 53, choose **Create record** under **Records** tab
- Select **CNAME** as **Record type**
- In **Record name** field, paste **CNAME name** from Certificate Manager (paste only the values before dot)
- In **Value** field, paste **CNAME value** from Certificate Manager (paste the whole values excluding the dot at the end)
- Leave other settings as default and choose **Create records**
- In CloudFront page, open the distribution, under **General** tab, choose **Edit**
- Enter the custom domain name in **Alternate domain name (CNAME) - _optional_** field and choose **Next**
- In **Custom SSL certificate - _optional_** dropdown, select the ACM certificate associated with the custom domain name
- Leave other settings as default and choose **Save changes**

## 6. Create Route 53 Alias Record for CloudFront Distribution
- In Route 53, select **Hosted zones** 
- Select **Create record** under **Records** tab
- Choose **A - Routes traffic to an IPv4 address and some AWS resources** as **Record type**
- Leave **Record name** blank and toggle on **Alias**
- Select **Alias to CloudFront Distribution** and choose the CloudFront distribution domain name
- Choose **Create records**

This creates an Alias A record that allows Route 53 to route traffic for the custom domain to CloudFront distribution without requiring an IP address


<img width="1834" height="747" alt="cname" src="https://github.com/user-attachments/assets/d5e3e366-5e55-45a2-8aef-739e61c838c1" />



## The static website is now live and securely accessible via a custom domain over HTTPS through CloudFront



 <img width="1840" height="960" alt="final page" src="https://github.com/user-attachments/assets/b8cfba09-6c96-44fc-876c-004d6bab95e0" />








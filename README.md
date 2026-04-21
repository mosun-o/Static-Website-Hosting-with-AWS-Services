# Static-Website-Hosting-with-AWS-Services

# Overview

A static website serves pre built, fixed content such as HTML, CSS, JavaScript, and media files without executing business logic or querying a database. An example is GitHub Pages.

This project demonstrates how **Amazon S3**, **Amazon CloudFront**, **Amazon Route 53**, and **AWS Certificate Manager (ACM)** integrate to host a fast, scalable, secure, and highly available static website with low latency global content delivery.

S3 stores the static website files and serves as the origin for the CloudFront distribution. CloudFront improves performance by caching content at edge locations worldwide and delivering it from the nearest location to users. Route 53 manages DNS resolution and routes user requests for the custom domain to the CloudFront distribution. ACM provisions and manages the SSL/TLS certificate used by CloudFront, enabling HTTPS and secure communication between clients (browsers) and the distribution.

# Architecture

<img width="904" height="316" alt="image" src="https://github.com/user-attachments/assets/e7ccc58e-d70d-4f41-9670-d44bf839cef1" />

## Request Flow
- **1.** Users access the static website using a custom domain from anywhere in the world
- **2.** The request first hits Route 53, which performs DNS resolution by mapping the custom domain to the associated CloudFront distribution (since CloudFront is the endpoint for the custom domain)
- **3.** The client (user's browser) establishes a secure HTTPS connection using the SSL/TLS certificate provided by Certificate Manager, and sends the request to the CloudFront distribution
- **4.** CloudFront receives the request at the nearest edge location. If the requested content is already cached (cache hit), CloudFront serves it immediately from the edge location, and if the content is not cached (cache miss), CloudFront forwards the request to the configured origin (S3 bucket) to retrieve the content
- **5.** Once CloudFront retrieves the content, it caches it at the edge location for future requests and returns the content to the user
- **6.** This process reduces latency by serving content closer to users, minimizes direct requests to the origin S3 bucket, improves performance, and enables secure, efficient global content delivery

Edge locations are globally distributed AWS data centers that cache content and act as Points of Presence (PoPs) for AWS services enabling data to be served from the location closest to the end user which significantly reduces latency and the distance data must travel




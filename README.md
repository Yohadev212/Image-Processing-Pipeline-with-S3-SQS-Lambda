# Image-Processing-Pipeline-with-S3-SQS-Lambda
Project 2: Serverless Image Processing Pipeline with S3, SQS &amp; Lambda
# Serverless Image Processing Pipeline with Amazon S3, SQS, Lambda & Step Functions

## Project Overview

This project demonstrates a fully serverless, event-driven image processing pipeline built on AWS. Users upload images to an Amazon S3 source bucket using pre-signed URLs. Upload events are sent to Amazon SQS, where AWS Lambda processes the images by validating, resizing, watermarking, and storing them in a destination S3 bucket. AWS Step Functions orchestrate the workflow, while CloudFront distributes processed images globally with low latency.

The architecture follows AWS Well-Architected Framework best practices by providing scalability, fault tolerance, high availability, and cost optimization without managing servers.

---

# Solution Architecture

```
                   +----------------------+
                   |      Client App      |
                   +----------+-----------+
                              |
                              | Upload Request
                              |
                    API Gateway (REST API)
                              |
                              |
                          AWS Lambda
                 (Generate Pre-Signed URL)
                              |
                              |
                      Amazon S3 Source Bucket
                              |
                    S3 Event Notification
                              |
                              |
                         Amazon SQS Queue
                              |
                              |
                       Lambda Image Worker
                              |
                    Starts Step Functions
                              |
          +-----------------------------------------+
          |                                         |
          |   Validate Image                        |
          |        ↓                                |
          |   Resize Image                          |
          |        ↓                                |
          |   Add Watermark                         |
          |        ↓                                |
          |   Extract Metadata                      |
          |        ↓                                |
          |   Store Processed Image                 |
          +-----------------------------------------+
                  |                    |
                  |                    |
                  |                    |
      Amazon S3 Destination      Amazon DynamoDB
             Bucket              Metadata Table
                  |
                  |
            Amazon CloudFront
                  |
                  |
              End Users

                     |
                     |
                 Amazon SNS
         Success / Failure Notifications

             Failed Messages
                   ↓
            Amazon SQS DLQ
```

---

# Project Objectives

- Build a fully serverless image processing system.
- Process uploaded images automatically.
- Resize and watermark images.
- Store image metadata.
- Deliver processed images globally.
- Implement fault-tolerant asynchronous processing.
- Demonstrate event-driven AWS architecture.

---

# AWS Services Used

| Service | Purpose |
|----------|----------|
| Amazon S3 | Store original and processed images |
| Amazon SQS | Decouple uploads from processing |
| Amazon SQS DLQ | Store failed processing events |
| AWS Lambda | Image processing and API logic |
| Lambda Layers | Package Pillow or Sharp libraries |
| AWS Step Functions | Orchestrate image workflow |
| Amazon DynamoDB | Store image metadata |
| Amazon CloudFront | Global image delivery |
| Amazon API Gateway | Generate pre-signed upload URLs |
| Amazon SNS | Job completion notifications |
| Amazon CloudWatch | Logging and monitoring |
| AWS IAM | Secure permissions |

---

# Architecture Components

## 1. API Gateway

Provides an endpoint that generates pre-signed upload URLs.

Responsibilities:

- Receive upload request
- Invoke Lambda
- Return secure upload URL

---

## 2. Lambda (Pre-Signed URL)

Generates temporary upload URLs without exposing AWS credentials.

Input:

- Filename
- File type

Output:

- Pre-signed URL

---

## 3. Amazon S3 Source Bucket

Stores original uploaded images.

Features:

- Versioning enabled
- Event Notifications
- Bucket Policy
- Lifecycle Rules
- Server-side encryption

---

## 4. Amazon SQS

Receives image upload events.

Benefits:

- Loose coupling
- Retry mechanism
- High durability
- Buffer traffic spikes

---

## 5. Dead Letter Queue (DLQ)

Stores failed image processing jobs after maximum retry attempts.

Benefits:

- Prevents message loss
- Simplifies troubleshooting
- Supports replay of failed jobs

---

## 6. AWS Lambda Image Processor

Triggered by SQS messages.

Responsibilities:

- Download image
- Validate format
- Resize image
- Apply watermark
- Save processed image
- Invoke Step Functions
- Update DynamoDB
- Publish SNS notification

Image libraries are packaged using Lambda Layers.

---

## 7. AWS Step Functions

Coordinates the processing workflow.

Workflow:

```
Validate Image
      ↓
Resize Image
      ↓
Apply Watermark
      ↓
Extract Metadata
      ↓
Store Image
      ↓
Update Database
      ↓
Send Notification
```

Benefits:

- Error handling
- Retry policies
- Visual workflow
- State management

---

## 8. Amazon DynamoDB

Stores metadata for each processed image.

Example Attributes:

| Attribute | Description |
|------------|-------------|
| ImageID | Unique identifier |
| FileName | Original filename |
| UploadTime | Upload timestamp |
| Width | Image width |
| Height | Image height |
| Status | Processing status |
| ProcessedImageURL | Destination URL |

---

## 9. Amazon S3 Destination Bucket

Stores processed images.

Features:

- Lifecycle Rules
- Versioning
- Encryption
- Optimized for CloudFront

---

## 10. Amazon CloudFront

Delivers processed images globally.

Benefits:

- Low latency
- Edge caching
- HTTPS
- Cost optimization

---

## 11. Amazon SNS

Publishes notifications.

Examples:

- Image processing completed
- Processing failed
- Workflow timeout

Subscribers may include:

- Email
- Lambda
- SMS
- HTTP endpoint

---

## 12. Amazon CloudWatch

Provides:

- Logs
- Metrics
- Dashboards
- Alarms

Metrics monitored:

- Lambda Errors
- Lambda Duration
- SQS Queue Depth
- Step Function Failures
- CloudFront Requests

---

# Processing Workflow

### Step 1

Client requests upload URL.

↓

### Step 2

API Gateway invokes Lambda.

↓

### Step 3

Lambda returns pre-signed URL.

↓

### Step 4

Client uploads image to S3.

↓

### Step 5

S3 publishes an event.

↓

### Step 6

Event is delivered to Amazon SQS.

↓

### Step 7

Lambda polls SQS.

↓

### Step 8

Lambda starts Step Functions.

↓

### Step 9

Step Functions perform:

- Validation
- Resize
- Watermark
- Metadata extraction
- Storage

↓

### Step 10

Metadata stored in DynamoDB.

↓

### Step 11

Processed image saved to destination bucket.

↓

### Step 12

SNS sends notification.

↓

### Step 13

CloudFront serves processed image.

---

# Security

- IAM least privilege
- S3 bucket policies
- HTTPS only
- CloudFront Origin Access Control (OAC)
- Server-side encryption (SSE-S3)
- DynamoDB encryption
- Lambda execution roles
- API Gateway authorization
- Pre-signed URLs with expiration

---

# Reliability

- SQS decouples services
- Dead Letter Queue
- Lambda retries
- Step Function retries
- CloudWatch alarms
- Durable object storage

---

# Scalability

The solution automatically scales using:

- AWS Lambda
- Amazon SQS
- Amazon S3
- Step Functions
- CloudFront
- DynamoDB On-Demand Capacity

No infrastructure management is required.

---

# Cost Optimization

The architecture follows a pay-as-you-go model.

Cost optimization features include:

- Serverless compute
- S3 Lifecycle Rules
- CloudFront caching
- On-demand DynamoDB
- Automatic scaling
- No idle servers

---

# S3 Lifecycle Policy Example

| Object | Transition |
|----------|------------|
| Original Images | Move to Standard-IA after 30 days |
| Processed Images | Move to Glacier after 90 days |
| Temporary Files | Delete after 7 days |

---

# Lambda Layers

External image libraries are packaged separately using Lambda Layers.

Examples:

- Pillow (Python)
- Sharp (Node.js)

Benefits:

- Smaller deployment packages
- Faster deployments
- Reusable dependencies

---

# Project Folder Structure

```
serverless-image-processing/
│
├── README.md
├── architecture/
│   └── architecture-diagram.png
│
├── lambda/
│   ├── presigned-url/
│   ├── image-processor/
│   └── layer/
│
├── step-functions/
│   └── workflow.json
│
├── infrastructure/
│   ├── cloudformation.yaml
│   └── terraform/
│
├── images/
│
└── docs/
```

---

# Future Improvements

- Face detection using Amazon Rekognition
- Automatic image tagging
- AI-powered image moderation
- Multiple output resolutions
- Image format conversion (PNG, WebP, AVIF)
- Custom watermark templates
- CDN invalidation automation
- Multi-region disaster recovery

---

# Learning Outcomes

By completing this project, you will gain practical experience with:

- Event-driven architectures
- Amazon S3 Event Notifications
- Amazon SQS and Dead Letter Queues
- AWS Lambda and Lambda Layers
- AWS Step Functions orchestration
- DynamoDB metadata storage
- API Gateway pre-signed URLs
- CloudFront content delivery
- SNS notifications
- CloudWatch monitoring
- IAM security best practices
- Serverless application design
- AWS Well-Architected Framework principles

---

# Conclusion

This project demonstrates how to build a scalable, highly available, fault-tolerant, and cost-efficient serverless image processing pipeline using core AWS services. It highlights modern cloud-native design patterns, including asynchronous messaging, workflow orchestration, event-driven processing, and global content delivery, making it an excellent portfolio project for the **AWS Certified Solutions Architect – Associate (SAA-C03)** certification and cloud engineering roles.

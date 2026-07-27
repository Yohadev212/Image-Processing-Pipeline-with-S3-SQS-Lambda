Project Details 
________________________________________
First: The Project Idea
Imagine you have a website like Canva, Facebook, Instagram, or even an online store where people upload product images.
When a user uploads an image, the website doesn't just save it.
It also does some processing, for example:
•	Make a thumbnail 
•	Add a watermark 
•	Get image information 
•	Save a new processed image 
•	Send a notification to the user 
All these steps happen automatically, and there is no server that we manage.
This is called a Serverless Image Processing Pipeline.
Second : what is a pipeline :
Means workflow every step is call the next step for example uploading images then validate then resize , watermark , metadata , store, notify user 
Third : Lambada serivces 
Using lambada service for low cost & working without servers or EC2 just apply the code withtin 15 min works after that Labada is close . 
________________________________________
Fourth : Architecture Components
•	Amazon API Gateway
•	AWS Lambda
•	Amazon S3
•	Amazon SQS
•	Amazon SQS Dead Letter Queue (DLQ)
•	AWS Step Functions
•	Amazon DynamoDB
•	Amazon SNS
•	Amazon CloudFront

Fifth : Workflow
•	User requests an upload URL.
•	API Gateway invokes a Lambda function.
•	Lambda generates a pre-signed S3 URL.
•	User uploads the image to the source S3 bucket.
•	S3 sends an ObjectCreated event to Amazon SQS.
•	Processor Lambda reads the queue message.
•	AWS Step Functions executes: 
•	Validate Image
•	Resize Image
•	Apply Watermark
•	Store Processed Image
•	Metadata is written to DynamoDB.
•	SNS sends success or failure notifications.
•	Processed images are served through CloudFront.
________________________________________
Finally : Learning Objectives
•	Understand event-driven architectures on AWS
•	Use Amazon SQS for decoupling services
•	Process images with AWS Lambda
•	Build workflows using AWS Step Functions
•	Store image metadata in DynamoDB
•	Implement failure handling with DLQ
•	Deliver content globally using CloudFront


Private Serverless Galleria — Full Project Documentation
Author: Ema Sena Architecture Type: AWS Serverless + CloudFront Secure Edge Architecture Status: Working Production Prototype

1. Project Overview
This project is a fully private serverless photo gallery platform built on AWS.
The system includes:
Secure authentication with Cognito
CloudFront CDN
Lambda@Edge authorization
Private uploader
S3 image storage
CloudFront cache invalidation
Secure image delivery
Fully serverless infrastructure
The application allows authenticated users to:
Log in securely
Upload photos
View private gallery images
Access images through CloudFront only
Automatically refresh CDN after upload

2. Final Live URLs
Gallery
https://d3lewuugzhwqxx.cloudfront.net/
Private Uploader
https://d3lewuugzhwqxx.cloudfront.net/upload

3. High-Level Architecture
Users   ↓ CloudFront Distribution   ├── Lambda@Edge Authentication   ├── / → Galleria   ├── /upload → Uploader   └── HTTPS + CDN  API Gateway   ├── Galleria Lambda   └── Uploader Lambda  S3 Buckets   ├── originals   ├── thumbnails   └── resized/full  Cognito   └── Authentication

4. AWS Services Used
Compute
AWS Lambda
Lambda@Edge
CDN
Amazon CloudFront
Authentication
Amazon Cognito
API Layer
Amazon API Gateway
Storage
Amazon S3
Security
IAM Roles
IAM Policies
Logging
CloudWatch Logs

5. CloudFront Distribution
Distribution ID
E2GB17O45KJQ3
Domain
https://d3lewuugzhwqxx.cloudfront.net

6. Bucket Architecture
Originals Bucket
ema-galleria-originals-resize-v5-1779154323
Purpose:
Stores original uploaded images
Security:
Fully private
Public access blocked
Verification:
aws s3api get-public-access-block \   --bucket ema-galleria-originals-resize-v5-1779154323
Result:
{   "PublicAccessBlockConfiguration": {     "BlockPublicAcls": true,     "IgnorePublicAcls": true,     "BlockPublicPolicy": true,     "RestrictPublicBuckets": true   } }

7. Authentication Architecture
Authentication Provider
Amazon Cognito
Login Flow
User requests /upload   ↓ CloudFront Lambda@Edge checks cookies   ↓ If not authenticated:   Redirect to Cognito Hosted UI   ↓ User logs in   ↓ Cognito redirects to /parseauth   ↓ Lambda@Edge exchanges code for tokens   ↓ JWT cookies stored   ↓ User can access private routes

8. CloudFront Behaviors


9. Lambda@Edge Authentication
Function Used
galleria-auth-edge-CheckAuthHandler
Behavior Association
Viewer Request

10. Uploader Lambda
Responsibilities
Serve uploader UI
Upload images to S3
Create CloudFront invalidation
Handle private upload API

11. Final Uploader Lambda Code
Main Routing Logic
if (   event.path.startsWith("/api/file/") ||   event.path.startsWith("/upload/api/file/") ) {   return fileRoute(event); } else {   return servePublic(event); }

Upload Key Extraction
let key = event.path   .replace("/upload/api/file/", "")   .replace("/api/file/", "");

Upload Route Fix
if (event.path === "/" || event.path === "/upload") {   return serveIndex(event); }

CloudFront Invalidation
await cloudfront.send(new CreateInvalidationCommand({   DistributionId: DISTRIBUTION_ID,   InvalidationBatch: {     CallerReference: `${Date.now()}-${key}`,     Paths: {       Quantity: 1,       Items: ["/*"]     }   } }));

12. Uploader Frontend
Upload API Fix
let uploadBaseUrl = '/upload/api/file/';
This was critical because CloudFront behavior uses:
/upload*

13. CloudFront Configuration
Uploader Behavior
Path Pattern
/upload*
Origin
uploader-origin
Allowed Methods
GET, HEAD, OPTIONS, PUT, POST, PATCH, DELETE
Cache Policy
CachingDisabled
Origin Request Policy
AllViewerExceptHostHeader

14. IAM Permissions
Required S3 Permissions
{   "Effect": "Allow",   "Action": [     "s3:PutObject",     "s3:DeleteObject"   ],   "Resource": "*" }

CloudFront Invalidation Permission
{   "Effect": "Allow",   "Action": [     "cloudfront:CreateInvalidation"   ],   "Resource": "*" }

15. IAM Role Discovery
Command
aws cloudformation describe-stack-resources \   --stack-name dev-galleria-emasena \   --query "StackResources[?ResourceType=='AWS::IAM::Role'].[PhysicalResourceId]" \   --output text
Result
dev-galleria-emasena-galleriaRole-GLCpFTYYte1y

16. Upload Testing
CLI Upload Test
curl -i -X POST \   --data-binary @"$HOME/file.jpg" \   "https://d3lewuugzhwqxx.cloudfront.net/upload/api/file/test.jpg"

17. CloudFront Invalidation
Manual Invalidation
aws cloudfront create-invalidation \   --distribution-id E2GB17O45KJQ3 \   --paths "/upload" "/upload/*"

18. CloudWatch Logging
Tail Logs
aws logs tail /aws/lambda/dev-uploader-emasena-uploader-r7tufXs3dKil \   --follow

19. Major Challenges Solved
Challenge 1 — Upload Path Broken
Symptom
Error uploading. Max upload size is ~4MB
Actual Cause
Wrong upload API path.
Uploader used:
basePath + 'api/file/'
But CloudFront behavior required:
/upload/api/file/
Fix
let uploadBaseUrl = '/upload/api/file/';

20. Challenge 2 — 414 CloudFront Error
Symptom
414 ERROR The request could not be satisfied.
Cause
CloudFront Function conflict + auth redirect loop.
Fix
Removed conflicting CloudFront Function
Used only Lambda@Edge auth
Cleared cookies
Reconfigured behaviors

21. Challenge 3 — /upload Returned Not Found
Symptom
{"message":"Not Found"}
CloudWatch Error
ENOENT: no such file or directory, open '/var/task/public/upload'
Cause
Uploader Lambda treated /upload as static file.
Fix
if (event.path === "/" || event.path === "/upload") {   return serveIndex(event); }

22. Challenge 4 — key is not defined
Error
ReferenceError: key is not defined
Cause
Key extraction block accidentally removed during sed modifications.
Fix
let key = event.path   .replace("/upload/api/file/", "")   .replace("/api/file/", "");

23. Challenge 5 — Environment Variables Missing
Error
aws: [ERROR]: argument --s3-bucket: expected one argument
Cause
CODE_BUCKET environment variable not exported.
Fix
export CODE_BUCKET=ema-galleria-code-1779145806 export DEST_BUCKET=ema-galleria-originals-resize-v5-1779154323 export USER=emasena

24. Deployment Process
Build
npm run build

Package
npm run package

Deploy
npm run deploy

25. Complete Deployment Script
export CODE_BUCKET=ema-galleria-code-1779145806 export DEST_BUCKET=ema-galleria-originals-resize-v5-1779154323 export USER=emasena  rm -f uploader.zip package.zip packaged-template.yml bundle.js  npm run build  zip -r package.zip bundle.js public  npm run package npm run deploy

26. Security Architecture
Security Layers
Layer 1
HTTPS only
Layer 2
CloudFront CDN
Layer 3
Lambda@Edge Authentication
Layer 4
Private S3 buckets
Layer 5
IAM least privilege

27. CI/CD Pipeline
GitHub Actions Workflow
name: Deploy Serverless Galleria  on:   push:     branches:       - main  jobs:   deploy:     runs-on: ubuntu-latest

28. AI Image Tagging Future Architecture
Proposed Architecture
Upload image → S3 originals bucket → Tagging Lambda → Amazon Rekognition DetectLabels → Save tags to DynamoDB → Searchable gallery

29. Recommended Future Features
Feature Ideas
Delete photo button
File picker upload
AI tagging
DynamoDB metadata
Search
Albums
Multi-user support
React frontend
Tailwind UI
Mobile optimization
Terraform/CDK
CI/CD
Watermarking
EXIF parsing
Face recognition

30. Final Production Architecture


<img width="1047" height="555" alt="image" src="https://github.com/user-attachments/assets/dadc0ceb-74b0-4482-807a-195503f5469f" />



31. Skills Demonstrated
AWS Skills
Lambda
Lambda@Edge
CloudFront
Cognito
API Gateway
IAM
S3
CloudWatch
Engineering Skills
Debugging distributed systems
CDN troubleshooting
Edge authentication
IAM permissions
API routing
Secure architecture
Infrastructure troubleshooting
Production deployment
CLI automation

32. Final Outcome
The project successfully achieved:
Fully private serverless gallery
Secure uploader
Authentication-protected CDN
Private S3 architecture
Dynamic uploads
Automatic CloudFront refresh
Production-grade AWS security
End-to-end serverless architecture

33<img width="1021" height="537" alt="Screenshot 2026-05-22 at 9 28 47 PM" src="https://github.com/user-attachments/assets/6c372228-a95a-43bc-9031-414ca294a164" />
. Portfolio Summary
This project demonstrates advanced AWS serverless engineering capabilities including:
Edge authentication
CDN security
API Gateway integration
Lambda architecture
CloudFront behaviors
Secure upload flows
IAM policy management
Production troubleshooting
AWS CLI automation
Cloud-native architecture


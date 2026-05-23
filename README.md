Private Serverless Galleria — Executive Project Summary
Project Title

Private Serverless Galleria
AWS Secure Edge-Based Photo Gallery Platform

Project Overview

Private Serverless Galleria is a fully serverless, cloud-native photo gallery application built on AWS using a secure edge architecture.

The platform was designed to provide:

Secure authenticated image access
Private image uploads
CDN-based image delivery
Fully private storage
Global scalability
Minimal infrastructure management

The application uses Amazon CloudFront, Lambda@Edge, Cognito authentication, API Gateway, Lambda, and private S3 buckets to create a production-style media delivery platform.

Main Project Goals
Functional Goals
Build a private photo gallery
Support authenticated uploads
Deliver images efficiently
Protect media from public access
Automatically refresh uploaded content
Technical Goals
Use AWS serverless services
Implement CDN architecture
Apply edge authentication
Use scalable object storage
Reduce operational overhead
Security Goals
Prevent direct S3 access
Enforce HTTPS-only communication
Secure upload endpoints
Implement JWT-based authentication
Apply IAM least privilege
Final Architecture
User Browser
   │
   ▼
CloudFront CDN
   │
   ├── Lambda@Edge Authentication
   ├── Gallery Route
   ├── Upload Route
   │
   ▼
API Gateway
   │
   ├── Galleria Lambda
   └── Uploader Lambda
   │
   ▼
Private S3 Buckets
AWS Services Used
Category	Service
CDN	Amazon CloudFront
Authentication	Amazon Cognito
Compute	AWS Lambda
Edge Security	Lambda@Edge
API Layer	Amazon API Gateway
Storage	Amazon S3
Security	IAM
Monitoring	CloudWatch Logs
Security Architecture
Authentication

Authentication is handled through Amazon Cognito and validated using Lambda@Edge before CloudFront serves protected content.

JWT Cookie Flow
User requests /upload
→ CloudFront intercepts request
→ Lambda@Edge validates JWT cookies
→ If invalid:
   Redirect to Cognito login
→ User authenticates
→ JWT cookies issued
→ Access granted
Private Storage Design

All S3 buckets are configured as fully private.

Security Controls
S3 Block Public Access enabled
No public bucket policies
HTTPS-only access
CloudFront-controlled delivery
IAM least privilege permissions
Upload Flow
Authenticated User
   ↓
Uploader Frontend
   ↓
API Gateway
   ↓
Uploader Lambda
   ↓
Private S3 Bucket
   ↓
CloudFront Invalidation
   ↓
Updated Gallery
CloudFront CDN Responsibilities

CloudFront was used as:

CDN layer
HTTPS endpoint
Authentication gateway
Edge authorization layer
Secure image delivery mechanism
CloudFront Behaviors
Path	Purpose
/	Main Gallery
/upload*	Private Uploader
/parseauth	Cognito callback
/signout	Logout
Key Technical Challenges Solved
1. Upload Route Failure
Problem

Uploads failed despite valid API responses.

Root Cause

Incorrect upload API path behind CloudFront behavior.

Fix
let uploadBaseUrl = '/upload/api/file/';
2. CloudFront 414 Errors
Problem

CloudFront returned redirect-loop errors.

Cause

Conflicting CloudFront functions and auth redirects.

Resolution
Removed conflicting function
Standardized Lambda@Edge auth
Reconfigured behaviors
3. Not Found Errors
Problem

Uploader returned:

{"message":"Not Found"}
Cause

Lambda attempted to serve /upload as static file.

Fix
if (event.path === "/" || event.path === "/upload") {
  return serveIndex(event);
}
4. Runtime Reference Errors
Problem
ReferenceError: key is not defined
Cause

Upload key extraction removed during modifications.

Fix
let key = event.path
  .replace("/upload/api/file/", "")
  .replace("/api/file/", "");
Deployment Workflow
Build
npm run build
Package
npm run package
Deploy
npm run deploy
CI/CD Pipeline

GitHub Actions pipeline designed for:

Git Push
→ Build
→ Package
→ Deploy
→ CloudFront Invalidation
AI Image Tagging — Future Expansion

Planned future architecture:

Upload Image
→ S3
→ Lambda Trigger
→ Amazon Rekognition
→ DynamoDB Metadata
→ Searchable Gallery
Skills Demonstrated
AWS Skills
Lambda
Lambda@Edge
CloudFront
Cognito
API Gateway
S3
IAM
CloudWatch
Engineering Skills
CDN troubleshooting
Edge authentication
Secure cloud architecture
Distributed systems debugging
Infrastructure automation
Production troubleshooting
Serverless architecture design
Final Outcome

The project successfully achieved:

Fully private serverless gallery
Secure uploader system
Authentication-protected CDN delivery
Private cloud storage
Automated CDN refresh
Production-grade AWS security
End-to-end serverless architecture
Live Application
Gallery

Private Serverless Galleria

Uploader

Private Uploader

Author

Ema Sena

GitHub Repository

Serverless Galleria Private Repository

Scalable Image Upload Service

A lightweight and scalable image upload backend built using Python FastAPI, AWS S3, and NGINX.
The system is fully stateless and does not require any database. Uploaded images are stored directly in Amazon S3, and the generated image URL is returned in the API response.

System Design
                    ┌────────────────────┐
                    │      Client        │
                    └─────────┬──────────┘
                              │
                              ▼
                    ┌────────────────────┐
                    │   NGINX : Port 80  │
                    │   Load Balancer    │
                    └─────────┬──────────┘
                              │
                 ┌────────────┴────────────┐
                 ▼                         ▼
        ┌────────────────┐        ┌────────────────┐
        │ FastAPI App 1  │        │ FastAPI App 2  │
        │   Port 5001    │        │   Port 5002    │
        └────────┬───────┘        └────────┬───────┘
                 │                         │
                 └────────────┬────────────┘
                              ▼
                    ┌────────────────────┐
                    │      AWS S3        │
                    │   Storage Bucket   │
                    └────────────────────┘

NGINX distributes incoming traffic between multiple FastAPI backend instances using round-robin balancing.
Each backend independently processes the upload request and stores the image inside an S3 bucket.

Since the application is stateless, no database or session storage is required.

Requirements
Python 3.11+
FastAPI
NGINX
AWS Account
S3 Bucket
IAM User with S3 permissions
Project Setup
1. Clone Repository
git clone https://github.com/your-username/scalable-image-upload.git
cd scalable-image-upload
2. Create Virtual Environment
Windows
python -m venv venv
.\venv\Scripts\activate
3. Install Dependencies
pip install -r requirements.txt
AWS S3 Configuration
1. Create S3 Bucket

Create a new bucket from AWS S3 Console.

Disable:

Block all public access
2. Add Bucket Policy
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadAccess",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::your-bucket-name/*"
    }
  ]
}

Replace:

your-bucket-name

with your actual bucket name.

Environment Variables

Create a .env file in project root.

AWS_ACCESS_KEY_ID=YOUR_ACCESS_KEY
AWS_SECRET_ACCESS_KEY=YOUR_SECRET_KEY
AWS_REGION=ap-south-1
S3_BUCKET_NAME=YOUR_BUCKET_NAME
Running Multiple Backend Servers
Backend Server 1

Open first PowerShell window:

cd D:\Cloud\scalable-image-upload
.\venv\Scripts\activate
uvicorn app.main:app --port 5001
Backend Server 2

Open second PowerShell window.

Before running, change:

SERVER_ID = "5001"

to:

SERVER_ID = "5002"

Then run:

cd D:\Cloud\scalable-image-upload
.\venv\Scripts\activate
uvicorn app.main:app --port 5002
NGINX Configuration

Open:

D:\nginx\conf\nginx.conf

Replace configuration with:

worker_processes 1;

events {
    worker_connections 1024;
}

http {

    upstream backend_servers {
        server 127.0.0.1:5001;
        server 127.0.0.1:5002;
    }

    server {
        listen 80;

        location / {
            proxy_pass http://backend_servers;
        }
    }
}
Start NGINX
cd D:\nginx
start nginx
API Testing
Upload Through NGINX
curl.exe -X POST "http://localhost/upload" -F "file=@D:\cloud\test.jpg"
Upload Directly to Backend
curl.exe -X POST "http://127.0.0.1:5001/upload" -F "file=@D:\cloud\test.jpg"
Verifying Load Balancing

Open browser:

http://localhost

Refresh multiple times.

You should see responses alternating between:

{
  "message": "Server 5001 is running"
}

and

{
  "message": "Server 5002 is running"
}

This confirms round-robin distribution is functioning correctly.

Sample Upload Response
{
  "server": "5001",
  "url": "https://your-bucket.s3.ap-south-1.amazonaws.com/abc123.jpg"
}
GitHub Actions CI Pipeline

The CI workflow executes on:

push
pull_request

Pipeline steps:

Install dependencies
Verify project build
Run validation/tests
Fail pipeline if any step fails
Why No Database?

The backend immediately uploads images to S3 and returns the generated file URL to the client.
Since image metadata is not stored locally, a database is unnecessary.

This makes the system:

Stateless
Lightweight
Horizontally scalable
Easy to deploy behind a load balancer

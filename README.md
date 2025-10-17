CloudLaunch

 Assesment  Summary:

CloudLaunch is a minimal demonstrator: a public static website plus private document storage. This repository contains:
- IAM policy JSON for "cloudlaunch-user"
- S3 bucket policy for the public website
- CloudFormation template to create "cloudlaunch-vpc" and networking resources
- Instructions and CLI snippets to create resources and to test permissions

 What I implemented

Task 1 — S3 

 Buckets created (names used in policy):
 A] "cloudlaunch-site-bucket" — static website hosting, public read.
Purpose: Public static website hosting.

Steps taken:
Go to S3 Console → Create bucket
Name: cloudlaunch-site-bucket
Region: (Europe  Stockholm eu-north-1)
Block all public access → Uncheck (to allow public access)
Acknowledged warning.
Uploaded a basic index.html and style.css .Properties → Static website hosting
Enabled “Static website hosting”
Choose  “Host a static website”
Index document: index.html
Saved.

Bucket Policy (Public Read Only):
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::cloudlaunch-site-bucket/*"
    }
  ]
}


B] "cloudlaunch-private-bucket" — private, accessible to "cloudlaunch-user" with GetObject and PutObject only.
Purpose: Private storage for files.
Access: Only via IAM user (not public).

Steps:
Create bucket cloudlaunch-private-bucket
Leave Block all public access enabled (✅)
No bucket policy required (IAM will handle access control)


  C] "cloudlaunch-visible-only-bucket" — private, "cloudlaunch-user" can ListBucket but not read objects.
INFO: Visible in bucket listings but contents inaccessible.

Steps taken:
Created bucket cloudlaunch-visible-only-bucket
Block public access enabled.


  Task 1: IAM
Create Custom IAM Policy
Policy Name:

CloudlaunchS3AccessPolicy

Policy JSON:{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ListAllThreeBuckets",
      "Effect": "Allow",
      "Action": "s3:ListBucket",
      "Resource": [
        "arn:aws:s3:::cloudlaunch-site-bucket",
        "arn:aws:s3:::cloudlaunch-private-bucket",
        "arn:aws:s3:::cloudlaunch-visible-only-bucket"
      ]
    },
    {
      "Sid": "GetObjectSiteBucket",
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::cloudlaunch-site-bucket/*"
    },
    {
      "Sid": "AccessPrivateBucketObjects",
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::cloudlaunch-private-bucket/*"
    },
    {
      "Sid": "DenyDeleteEverywhere",
      "Effect": "Deny",
      "Action": "s3:DeleteObject",
      "Resource": [
        "arn:aws:s3:::cloudlaunch-site-bucket/*",
        "arn:aws:s3:::cloudlaunch-private-bucket/*",
        "arn:aws:s3:::cloudlaunch-visible-only-bucket/*"
      ]
    }
  ]
}


Explanation:
 ListBucket → user can see all three buckets in S3 list.
GetObject on site bucket → can download website files.
GetObject + PutObject on private bucket → can upload/read but not delete.
Explicitly denies delete anywhere.
No GetObject on visible-only bucket → can list but not read content.
Attached this policy to cloudlaunch-user.



     S3 site URL: `S3_WEBSITE_ENDPOINT`Z [https://cloudlaunch-site-bucket.s3.eu-north-1.amazonaws.com/cloudlaunch.html] 
**CloudFront URL (optional)**: `CLOUDFRONT_URL` (https://d1wwjrew4ozcr7.cloudfront.net/cloudlaunch.html)

Security note: No credentials or private keys are stored in this repo.

   Task 2 — VPC design
- Created a VPC "cloudlaunch-vpc" with CIDR "10.0.0.0/16".
- Subnets:
  - Public Subnet: 10.0.1.0/24 (for load balancers)
  - App Subnet: 10.0.2.0/24 (private)
  - DB Subnet: 10.0.3.0/28 (private)
  
- Internet Gateway "cloudlaunch-igw" attached to VPC
- Route tables:
  - "cloudlaunch-public-rt" → 0.0.0.0/0 to IGW (associated with Public Subnet)
  - "cloudlaunch-app-rt" (no 0.0.0.0/0)
  - "cloudlaunch-db-rt" (no 0.0.0.0/0)
  
- Security groups:
  -  cloudlaunch-app-sg : allow inbound TCP 80 from  10.0.0.0/16 
  -  cloudlaunch-db-sg : allow inbound TCP 3306 from  10.0.2.0/24 

CloudFormation file: "cloudlaunch-vpc.yml"

       Files in this repo
- `README.md` (this [file)](url)
- `cloudlaunch-user-policy.json` (IAM policy JSON)
- `s3-site-bucket-policy.json` (public website bucket policy)
- `cloudlaunch-vpc.yml` (CloudFormation template)


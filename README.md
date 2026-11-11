# Cloud Learning Log

Daily hands-on AWS and DevOps labs. Every day follows the same loop:

1. **Problem**: a real scenario to solve
2. **Build**: in my own AWS account
3. **Break it on purpose**: then fix it from the error messages
4. **Explain**: what I can now explain without notes

## Log

| Date | Topic | Highlights |
|---|---|---|
| 2026-10-26 | [VPC fundamentals](2026-10-26-vpc-day-1.md) | Custom VPC 10.0.0.0/16, route tables vs security groups, 3 protected defaults |
| 2026-10-27 | [Subnets in two AZs](2026-10-27-vpc-day-2.md) | 4 subnets across 2 AZs, CIDR math, overlap and out-of-range breaks |
| 2026-10-28 | [Route tables and the internet gateway](2026-10-28-vpc-day-3.md) | Public route table, IGW, longest prefix match, removed 0.0.0.0/0 on purpose |
| 2026-10-29 | [NAT gateway and private subnets](2026-10-29-vpc-day-4.md) | NAT in public subnet, packet path, IGW vs NAT, deleted same day (~3 cents) |
| 2026-10-30 | [Security groups vs network ACLs](2026-10-30-vpc-day-5.md) | web-sg (80/443, no SSH), stateful vs stateless, ephemeral ports, broken NACL built for next week |
| 2026-11-03 | [CloudWatch + CloudTrail](2026-11-03-cloudwatch-cloudtrail.md) | Found a real bug via 0 Lambda invocations; tested an alarm to SNS email; built a dashboard; traced a deploy and a security group change in CloudTrail; created a trail to S3 |
| 2026-11-04 | [AWS Config](2026-11-04-aws-config.md) | Recorded security groups; restricted-ssh rule caught SSH open to 0.0.0.0/0 (Noncompliant), fixed it back to Compliant; read the resource timeline and JSON diff; detective vs preventive controls |
| 2026-11-05 | [Project 3: EC2 web server](2026-11-05-project-3-ec2-web-server.md) | EC2 in deo-lab-vpc with user data, Session Manager (no SSH/keys), CPU alarm to SNS triggered for real, broke and fixed a NACL (port 80 + 1024–65535 reply ports) |
| 2026-11-09 | [S3 versioning + lifecycle](2026-11-09-s3-versioning.md) | Versioned bucket, recovered a deleted file by removing its delete marker, lifecycle rule for noncurrent versions (30 days), broke it by suspending versioning (null version overwrote version 1) |
| 2026-11-10 | [S3 storage classes + encryption](2026-11-10-s3-storage-classes-encryption.md) | Uploaded objects to Standard-IA and Glacier, lifecycle transitions (30/90/365 days), restored an archived object, lifecycle vs Intelligent-Tiering, SSE-S3 vs SSE-KMS |
| 2026-11-11 | [DynamoDB PITR + deletion protection](2026-11-11-dynamodb-pitr-restore.md) | Enabled point-in-time recovery on the live Inquiries table, deleted a test record and recovered it via restore to a new table, copied it back, enabled deletion protection |

## Projects built along the way
- [Serverless Photography Website](https://github.com/deokanonga/serverless-photography-website)
- [Serverless Client Tracker](https://github.com/deokanonga/serverless-client-tracker)


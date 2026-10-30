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


## Projects built along the way
- [Serverless Photography Website](https://github.com/deokanonga/serverless-photography-website)
- [Serverless Client Tracker](https://github.com/deokanonga/serverless-client-tracker)


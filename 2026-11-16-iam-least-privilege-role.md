# CloudOps prep day 1: IAM review, a least-privilege role for the gallery

## The problem
The gallery Lambda (built later) must read the client photos to sign links, but it must never write, delete or touch other folders. I built the exact role it will use and tried to break it.

## What I built
| Piece | Name | Settings |
|---|---|---|
| Policy | gallery-read-only | Allow s3:GetObject on arn:aws:s3:::deo-gallery-4827/originals/smith-wedding/* |
| Role | deo-gallery-reader | Trusted entity: AWS service Lambda; policy gallery-read-only attached |
| Test | IAM Policy Simulator | GetObject Allowed, PutObject Denied |

## Role vs policy
| Part | Question it answers | In my lab |
|---|---|---|
| Trust policy | WHO may assume the role (sts:AssumeRole) | lambda.amazonaws.com |
| Permissions policy | WHAT the role may do, on which resources | GetObject on one folder |
The role itself is permanent; each Lambda run gets fresh temporary credentials from STS.

## Deny types and layers
| Term | Meaning |
|---|---|
| Implicit deny | No statement says Allow, so the answer is no (my PutObject) |
| Explicit deny | A statement says Deny; beats every Allow |
| SCP | Guardrail from the AWS Organization; only sets the maximum, never grants |
Final permission = SCP allows AND IAM policy allows. Order: explicit Deny, then any Allow, else implicit deny.

## Breaks and wrong guesses
| What I thought | What is true | Why |
|---|---|---|
| The role is temporary, a new one per Lambda event | One permanent role, temporary credentials per run | STS issues a short-lived token each time |
| "Denied" because no Lambda exists yet | The simulator tests the role directly. The deny said "Denied by AWS Organizations" | My account is a member of an Organization I do not manage; I cannot read its SCPs. With the SCP toggle off, the policy alone gives Allowed |
| Roles control who has access | Trust policy controls who; permissions policy controls what | Two separate documents on the role |

## What I learned
- ARN (Amazon Resource Name) is for IAM policies and every AWS service; S3 URI (s3://...) is for S3 commands. ARN pattern: arn:aws:s3:::bucket/path
- A policy statement needs Effect, Action, Resource; it never names who
- A folder in S3 exists only when a file is inside it (previews/ did not exist yet)
- The simulator tests permissions without touching anything

## Teach-back: 4 / 4
Small fix: the ARN starts arn:aws:s3::: (three colons).

## Weak-topic list for CloudOps prep
- SCPs (ceiling, never grant)
- Trust policy vs permissions policy
- Implicit vs explicit deny

## Cleanup
- Kept: policy gallery-read-only and role deo-gallery-reader (free; the gallery Lambda will use them)

## I can explain
- The difference between a trust policy and a permissions policy
- Why a role is permanent but its credentials are temporary
- Why a request with no matching Allow is denied, and how an SCP limits it

## Next
- Tue Nov 17: networking review (rebuild a VPC diagram from memory)

# CloudOps prep day 7: IAM lab (trust policy vs permissions policy, explicit deny)

## The problem
On the readiness test I missed three IAM questions: trust policy vs permissions policy, explicit deny, and which CloudTrail field shows the person on a shared role. I wanted to see each one fail and work with my own eyes.

## What I built
| Piece | Setting |
|---|---|
| Role | deo-lab-trust-test |
| Trust policy | Principal = this account, action sts:AssumeRole |
| Permissions | gallery-read-only (GetObject on one photo folder) |
| Test place | CloudShell, assume the role with the CLI and use its temporary keys |

## Steps
| Step | What I did | Why |
|---|---|---|
| 1 | Created the role (trust: this account, permission: gallery-read-only) | Two separate policies on one role |
| 2 | Assumed the role with sts assume-role, session name deo-test | Becoming the role = what the trust policy allows |
| 3 | Exported the 3 temporary keys, checked get-caller-identity | Proof that I am the role, not myself |
| 4 | Read a photo with s3 cp (worked), listed the folder with s3 ls (denied) | GetObject was allowed, ListBucket was not |
| 5 | Break 1: trust policy changed to Lambda only, tried to assume again | AccessDenied on sts:AssumeRole: I am not on the WHO list |
| 6 | Break 2: added an inline Deny on GetObject, read the photo again | Explicit deny beats the Allow |
| 7 | Cleaned up: unset the keys, deleted the downloaded files, deleted the lab role | Nothing left running |

## Breaks and wrong guesses
| What I thought | What is true | Why |
|---|---|---|
| Allow wins over deny | An explicit Deny always wins | Order: explicit Deny, then Allow, otherwise implicit deny |
| ROLE_ARN is a function | It is a variable | A labeled box holding the role's address |
| --output text does something to the query | --query picks the field, --output picks the format | Text output has no quotes, needed inside variables |
| A 403 means the policy denied me | A 403 has many causes | Scrambled keys, wrong file name (no ListBucket gives 403 for a missing file) |
| The Deny test failed | It was never tested | My exports were scrambled, so I was still my admin identity; get-caller-identity first |
| Error wording is the same for every deny | Two different sentences | "no identity-based policy allows" = implicit deny; "with an explicit deny" = a Deny statement matched |

## What I learned
- Trust policy = WHO may become the role. Permissions policy = WHAT the role may do
- Assume-role error "not authorized to perform sts:AssumeRole" means fix the trust policy
- Explicit deny wins over every allow
- Always run get-caller-identity before a test, so you know who you are
- Variable names must match field names exactly (AccessKeyId, SecretAccessKey, SessionToken)
- S3 names are case-sensitive; no list permission turns a missing file into a 403
- In CloudTrail the person on a shared role is the session name at the end of userIdentity arn (here deo-test); userIdentity type only says AssumedRole

## Cleanup
- Unset the temporary keys, removed the downloaded copies
- Lab role and its inline Deny deleted; gallery-read-only and deo-gallery-reader kept

## I can explain
- Who may become a role (trust) vs what it can do (permissions)
- Why an explicit deny beats an allow, and how the two error sentences differ
- How I know which identity I am in the CLI

## Next
- Wed Nov 25: Config rules and CloudTrail fields
- Fri Nov 27: practice test 3, then book Dec 1 (80% or more) or Dec 8

# Project day: client gallery v1, private S3 bucket and presigned URLs

## The problem
Client photos must stay private, but one client must be able to open their own photos without an AWS account. In Google Drive that is "share". In S3 the bucket stays locked and the owner hands out a signed link that expires.

## What I built
| Piece | Name | Settings |
|---|---|---|
| Bucket | deo-gallery-4827 | us-east-2, Block Public Access ON, SSE-S3 |
| Test gallery | originals/smith-wedding/ | 3 photos (moved in with S3 Move: copy + delete) |
| Previews | previews/smith-wedding/ | Small copies, stay in Standard |
| Presigned link | IMG_2068.JPG | `aws s3 presign s3://deo-gallery-4827/originals/smith-wedding/IMG_2068.JPG --expires-in 900` |
| Lifecycle rule | originals-to-deep-archive | Prefix originals/, Deep Archive after 365 days |

## The tests
| Test | Result | Reading |
|---|---|---|
| Plain object URL in incognito | AccessDenied | No signature: the lock works |
| Presigned link in incognito | Photo opens | The signature proves the owner allowed it |
| Changed key to IMG_2069 after 15 min | AccessDenied, "Request has expired" | Expiry is checked first (signed 01:56:26 UTC, expired 02:11:26) |
| Original link after 15 min | Expired | The key card dies on schedule |

## Breaks and wrong guesses
| What I thought | What is true | Why |
|---|---|---|
| An IAM role per client, photos in a public subnet | Neither | Clients have no AWS account, S3 has no subnets. Access = Block Public Access + presigned links |
| Changing one letter gives a signature error | I got "expired" | S3 checks the time first; the same AccessDenied code hides 3 reasons, read the Message |
| One link can cover a whole album | No | A presigned URL signs exactly one object |
| Archived photo opens at once | No | Deep Archive objects must be restored first (about 12 h), until then InvalidObjectState |

## What I learned
- Folders in S3 are only part of the key: originals/smith-wedding/IMG_2068.JPG
- S3 has no real move: move = copy + delete (and no real rename)
- The error Code can be the same; the Message line tells the cause
- CloudShell uses temporary credentials (X-Amz-Security-Token), so a link can die earlier than its expiry
- Real gallery = HTML/CSS page + Lambda that checks a password, lists the album and signs a link for every photo; or CloudFront signed cookies that cover a whole path
- Help yourself: `aws s3 help` and `aws s3 presign help` (q to leave)

## Teach-back: 3 / 5
Weak spots: Block Public Access is ON (I said off); the third AccessDenied reason is a wrong signature; how the page shows all photos (Lambda signs each, or signed cookies); S3 vs EBS vs EFS needs more than a label.

## Cleanup
- Kept: bucket deo-gallery-4827 with the test gallery and lifecycle rule (cents per month)

## I can explain
- Why a private bucket plus a presigned URL beats making photos public
- How to tell AccessDenied from no signature, wrong signature and expired link
- Why a Deep Archive photo needs a restore before it can be downloaded

## Next
- Mon Nov 16: CloudOps Engineer Associate prep starts; gallery page, Lambda and CloudFront in the labs
- Weekend: Python Essentials modules on your own

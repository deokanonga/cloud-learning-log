# Storage day 2: S3 storage classes, lifecycle transitions and encryption

## The problem
A photographer keeps every wedding's original files. They are opened a lot in the first weeks, then almost never, but they must not be lost. Paying the "open it any second" price for files nobody opens is wasted money.

## What I built
| Piece | Name | Settings |
|---|---|---|
| Lab bucket | deo-lab-versioning-4827 | Same bucket as day 1, us-east-2 |
| Object in a cheaper class | wedding-2025.txt | Uploaded directly as Standard-IA |
| Archived object | wedding-2023.txt | Uploaded directly as Glacier Flexible Retrieval |
| Lifecycle rule | archive-by-age | Current versions: Standard-IA at day 30, Glacier Flexible Retrieval at day 90, Glacier Deep Archive at day 365 |
| Restore | wedding-2023.txt | Initiate restore, 1 day, Expedited tier |

## Storage classes
| Class | Get the file back | Catch |
|---|---|---|
| Standard | Instantly | Highest storage price |
| Standard-IA | Instantly | Cheaper storage, fee per read, 30 days minimum |
| Glacier Instant Retrieval | Instantly | Cheaper again, higher read fee, 90 days minimum |
| Glacier Flexible Retrieval | 1-5 min (Expedited), 3-5 h (Standard), 5-12 h (Bulk) | Needs a restore first |
| Glacier Deep Archive | About 12 h (Standard), up to 48 h (Bulk) | Cheapest, needs a restore, 180 days minimum |

The storage class belongs to each object, not to the bucket. One bucket can hold files on different shelves.

## Two lifecycle rules, two jobs
| Rule | Acts on | What it does |
|---|---|---|
| clean-old-versions | Noncurrent (old) versions | Permanently deletes them after 30 days |
| archive-by-age | Current (live) versions | Moves them to cheaper classes at day 30, 90 and 365 |

They do not conflict because they never touch the same thing. Both have the scope "Entire bucket", so they apply to every file, including hello.txt from day 1.

## Lifecycle rule vs Intelligent-Tiering
| | Lifecycle rule | S3 Intelligent-Tiering |
|---|---|---|
| Decides by | Age since upload | Whether the file was accessed |
| File opened on day 29 | Still moves on day 30 | Timer resets, stays on the fast tier |
| Best when | I know the pattern | I cannot predict the pattern |

## Encryption
- Every new S3 object is encrypted automatically with SSE-S3 (keys managed by S3, free)
- Bucket > Properties > Default encryption shows the rule for new files. The object's own Properties show how that file was encrypted
- "Do not specify an encryption key" at upload means "use the bucket default", not "no encryption"
- SSE-KMS uses a key I control in KMS: a second lock, with every use logged in CloudTrail
- Storage class and encryption are separate choices: the class is the shelf, the encryption is the lock

## Breaks
| What I tried | What happened | Why |
|---|---|---|
| Opened wedding-2023.txt in Glacier | Open was greyed out | An archived object cannot be read until it is restored |
| Initiated a restore (Expedited, 1 day) | "Restoration complete", file opens, Storage class still says Glacier | A restore makes a temporary copy; the object stays in Glacier |

## What I learned
- Days in a lifecycle rule are counted from the upload day, not from the previous move
- To find the class on a date: turn the date into days since upload, then take the last transition below that number
- Each transition costs a small fee per object, and S3 skips objects under 128 KB by default
- The saving comes from matching the class to how fast I need the file back
- A scope filter (prefix or tag) limits a rule to part of a bucket

## Teach-back: 3.5 / 4
- Weak: Standard and Standard-IA are equally fast. IA is cheaper to store but charges a fee each time a file is read

## Cleanup
- Nothing to delete. Lab bucket kept for the rest of storage week (a few bytes)

## Also today
- Passed Introduction to Linux (LFS101) from The Linux Foundation and added it to LinkedIn
- Started planning a client photo gallery on S3 that uses these lifecycle rules for old galleries

## I can explain
- When to pick Standard, Standard-IA or a Glacier class
- Why two lifecycle rules on one bucket do not conflict
- What a Glacier restore is and why the file is still in Glacier afterwards
- The difference between SSE-S3 and SSE-KMS

## Next
- Storage day 3: backups and point-in-time recovery for the Inquiries table

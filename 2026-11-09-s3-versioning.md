# Storage day 1: S3 versioning and lifecycle rules

## The problem
A website lives in an S3 bucket. One bad upload overwrites a good `index.html`, or someone deletes a photo by mistake. Without versioning, S3 keeps no copy and the file is gone.

## What I built
| Piece | Name | Settings |
|---|---|---|
| Lab bucket | deo-lab-versioning-4827 | us-east-2, Block all public access on, Bucket Versioning enabled |
| Test object | hello.txt | Uploaded 4 times with different text (version 1 to version 4) |
| Lifecycle rule | clean-old-versions | All objects; permanently delete noncurrent versions after 30 days; delete expired object delete markers |

The test file was made in the Mac Terminal:
```
cd ~/Desktop
echo "version 1" > hello.txt
```
`echo` prints the text, `>` sends it into the file and replaces what was there. Each file is 10 bytes: 9 characters plus the new line `echo` adds.

## How versioning behaves
| Action | What S3 really does | Can I undo it? |
|---|---|---|
| Upload a file with the same name | Keeps the old one as a noncurrent version, the new one becomes current | Yes |
| Delete with Show versions off | Deletes nothing. Adds a delete marker (0 B) on top that hides the file | Yes: delete the delete marker |
| Delete a specific version with Show versions on | Removes that version for good ("permanently delete") | No |
| Delete the delete marker | The newest remaining version becomes current again | - |

## Two different switches
| Switch | Where | What it does |
|---|---|---|
| Bucket Versioning | Bucket > Properties | Decides whether S3 keeps old copies (Enabled / Suspended) |
| Show versions | Bucket > Objects, above the list | Only changes what the console displays |

## Breaks
| What I tried | What happened | Why |
|---|---|---|
| First upload before versioning was on | That version got Version ID `null` | `null` is the ID of anything uploaded while versioning is off or suspended |
| Deleted hello.txt the normal way | File vanished from the list, all 3 versions still there | A normal delete only adds a delete marker |
| Suspended versioning, uploaded version 4 | Only 3 rows left: version 1 was gone for good | The new upload also got ID `null`, and a file name can hold only one `null` version, so it overwrote version 1 |

## What I learned
- Versioning only protects from the moment it is enabled. It does not bring back what was overwritten before
- Versioning can never be turned fully off again, only suspended
- Suspended keeps the existing history but brings back "overwrite means lost" for new uploads
- Every kept version is billed like a normal object, so versioning needs a lifecycle rule
- "Noncurrent" means every version that is not the live one. "Expire current versions" is the dangerous checkbox, it removes live files
- A lifecycle rule runs in the background about once a day. It does not act the moment I upload
- With 30 days for noncurrent versions, a file overwritten on Dec 1 can be recovered until about Dec 31
- "Number of newer versions to retain" always keeps the last N old versions as spares, whatever their age
- A lifecycle rule is not an IAM role. Rule = automatic cleaner, role = permissions

## Teach-back: 3 / 4
- Weak: a normal delete does not remove the selected file either. It removes nothing and only adds a delete marker
- Weak: I mixed up the Show versions toggle (display) with the Bucket Versioning setting (the real switch)

## Cleanup
- Re-enabled Bucket Versioning
- Kept the bucket for the rest of storage week (a few bytes, no cost)

## Also today
- Added automatic dark mode to www.deokanonga.com with CSS variables and `prefers-color-scheme`, deployed and working

## I can explain
- What a delete marker is and how to undelete a file in S3
- Why Version ID `null` appears and why suspending versioning is risky
- Why a versioned bucket needs a lifecycle rule

## Next
- Storage day 2: S3 storage classes and encryption

# Storage day 4: AWS Backup and backup plans

## The problem
PITR only covers 35 days and only DynamoDB. If bad data goes unnoticed for 2 months, PITR cannot go back that far. I want fixed copies kept in a separate place, with a rule for how often and how long.

## What I built
| Piece | Name | Settings |
|---|---|---|
| Vault | deo-lab-vault | Default key (aws/backup) |
| Plan | daily-inquiry-backup | Rule daily-35-days, daily, default window (5:00 AM UTC = 9 PM PT), keep 35 days |
| Assignment | inquiries-table | DynamoDB table Inquiries, default role |
| On-demand backup | Inquiries, "Backup now" at 17:41 PT | Kept 7 days (deletes itself Nov 19) |
| Restore | Inquiries-backup-restored | New table, 10 items, then deleted |

## Vault, plan, assignment
| Piece | Job |
|---|---|
| Vault | Stores the backups (recovery points). Does not protect resources |
| Plan | Schedule + how long to keep each backup |
| Assignment | Which resources the plan backs up (attached to the plan, not the vault) |

## Backup plan vs PITR
| | PITR | AWS Backup plan |
|---|---|---|
| Works | Continuously, every change logged | One fixed photo per schedule |
| Restore to | Any second in the last 35 days | Only the moment of that backup |
| Stored | Inside DynamoDB, not visible in my S3 | In my vault, I can see it |
| Use for | "I deleted an inquiry 10 minutes ago" | "I want a copy as it was on the 1st of the month" |

## Breaks and wrong guesses
| What I thought | What is true | Why |
|---|---|---|
| Vault protects the resources | The vault only stores backups | Resources are assigned to the plan |
| Protected resources lists my table | Empty until a first backup exists | It lists resources that already have a backup |
| Creating the backup means the table was deleted | The table is untouched | A backup is a sealed copy; restore turns it into a usable table, for a loss or just a test |
| The plan backs up before I assign anything | No | A plan with no assignment protects nothing |

## What I learned
- Retention (7 / 35 days) is how long AWS keeps a backup before deleting it; it drives cost
- "Backup now" takes the photo at that moment; anything added later is not in it
- The service gets a default role that lets it read the table and write to the vault
- A restore returns the data, not the settings: PITR, deletion protection, alarms and tags are off
- A backup you never test is only a hope; the restore proved mine works

## Teach-back: 3.5 / 4
Weak spot: question 3 said why the name must be new, but missed what the restored table does not have (PITR, deletion protection, alarms, tags).

## Cleanup
- Deleted Inquiries-backup-restored
- Kept: plan, assignment, vault and the 7 day on-demand backup (a few cents a month)

## I can explain
- The difference between a vault, a plan and an assignment
- When to use PITR and when AWS Backup
- Why a restore always builds a new table

## Next
- Friday: first version of the client photo gallery on S3 (own thread)
- Mon Nov 16: CloudOps Engineer Associate prep starts

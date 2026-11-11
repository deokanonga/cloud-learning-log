# Storage day 3: DynamoDB point-in-time recovery and deletion protection

## The problem
Every wedding inquiry from my website lives in one DynamoDB table, `Inquiries`. One wrong click in the console or one bug in a Lambda and a client's inquiry is gone, with no way to get it back.

## What I built
| Piece | Name | Settings |
|---|---|---|
| Point-in-time recovery | Inquiries | On, 35 day recovery period |
| Test record | PITRtest inquiry | Sent through the real website form, then deleted on purpose |
| Restore | Inquiries-restored | Restored to a chosen time (12:09), new table, entire table |
| Recovery | Inquiries | Copied the lost item back with JSON view and Create item |
| Deletion protection | Inquiries | On |

## The recovery, step by step
| Time (PT) | What happened |
|---|---|
| 11:56 | Turned on PITR. Earliest and latest restore point were both 11:56 |
| 12:06 | Test inquiry created through the site (`created_at` shows 20:06 UTC) |
| 12:08 | Scan showed 11 items, test item present |
| after 12:11 | Deleted the test item: 10 items |
| 12:09 | The restore time I chose, a moment I knew the data was good |
| about 12:30 | Inquiries-restored became Active with 11 items |
| after | Copied the item back into Inquiries, deleted the restored table |

## Deletion protection vs PITR
| | Deletion protection | PITR |
|---|---|---|
| Protects | The table itself | The items inside it |
| Type | Preventive: blocks "Delete table" | Recovery: rebuilds the table as it was at a chosen second |
| Cannot help with | A deleted or changed item | Does not stop any mistake from happening |

A table with real data needs both.

## S3 versioning vs DynamoDB PITR
| | S3 versioning | DynamoDB PITR |
|---|---|---|
| Works on | One object at a time | The whole table at once |
| Undo | Remove that object's delete marker | Restore the full table to a new table, then fetch what is needed |

## Breaks and wrong guesses
| What I thought | What is true | Why |
|---|---|---|
| With PITR on I can go back to yesterday | No | History starts the second PITR is enabled. "35 days" is how far back it can go once it has run that long |
| Use "Latest" as the restore time | Wrong choice after an accident | Latest is as close to now as possible, so it can be after the delete. Pick a time when the data was known to be good |
| The restore fixes my table | No | A restore always creates a new table. The live table is unchanged until I copy the data back |

## What I learned
- A restore never overwrites. It builds a second table, and takes several minutes even for 11 items
- I do not need to know when the accident happened, only a time when the data was still correct
- PITR does not track deleted items. It answers "what did the whole table look like at this second?"
- A restored table comes without PITR, deletion protection, alarms and tags
- DynamoDB tables cannot be renamed. To replace a broken table: point the application at the restored one, or delete and restore again under the original name
- Deleting the original and keeping the copy would lose everything written after the restore time
- `created_at` in the table is UTC, 8 hours ahead of Pacific time

## Teach-back: 4 / 4

## Cleanup
- Deleted Inquiries-restored and the test item
- Kept: PITR on and deletion protection on for Inquiries (a few cents a month)

## I can explain
- How to recover a deleted DynamoDB item with point-in-time recovery
- Why a restore is only half of a recovery
- The difference between preventing a mistake and recovering from one

## Next
- Storage day 4: AWS Backup and backup plans
- Friday: first version of the client photo gallery on S3

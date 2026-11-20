# CloudOps prep day 4: storage and resilience, recover a deleted file from an EBS snapshot

## The problem
A server's data disk can lose a file or get corrupted. I need to know which storage type is which, and how to bring data back. I built a small server with an extra disk, deleted a file on purpose and recovered it from a snapshot.

## What I built
| Piece | Name | Settings |
|---|---|---|
| Server | deo-lab-storage | t3.micro, Amazon Linux 2023, in deo-lab-public-a, no key pair, Session Manager |
| Data disk | deo-lab-data | EBS gp3, 1 GiB, same Availability Zone as the server, attached as /dev/sdf (shows as nvme1n1 inside Linux) |
| Snapshot | deo-lab-snap1 | Photo of the data disk with clients.txt on it |
| Restored disk | deo-lab-restored | New volume made from the snapshot, mounted on /restore |

## Three kinds of storage
| Type | What it is | Scope | In my labs |
|---|---|---|---|
| S3 | Object storage, files reached by key over the web | A whole Region (spread across several AZs) | Gallery bucket, presigned links |
| EBS | Block storage: the disk of one server | One Availability Zone, attached to one server | Root disk, deo-lab-data |
| EFS | File storage: a shared network folder | Many AZs, many servers mount it | Concept only |

## The recovery, step by step
| Step | Command or action | What it does |
|---|---|---|
| 1 | lsblk | Lists the disks the server sees (block devices) |
| 2 | sudo mkfs -t xfs /dev/nvme1n1 | Formats the NEW empty disk with a filesystem (erases it) |
| 3 | sudo mkdir /data and sudo mount /dev/nvme1n1 /data | Connects the disk to a folder |
| 4 | echo "..." \| sudo tee /data/clients.txt | Writes a file as administrator |
| 5 | Create snapshot of deo-lab-data | Photo of the disk, stored in S3 by AWS |
| 6 | sudo rm /data/clients.txt | The mistake on purpose |
| 7 | Create volume from the snapshot, same AZ, attach | A new disk with the old content |
| 8 | sudo mkdir /restore and sudo mount -o nouuid /dev/nvme2n1 /restore | Mounts the copy beside the original |
| 9 | sudo cp /restore/clients.txt /data/clients.txt | Copies only the lost file back |
| 10 | sudo umount /restore, terminate the server, delete both volumes and the snapshot | Clean up |

## Breaks and wrong guesses
| What I thought | What is true | Why |
|---|---|---|
| A new volume says "in-use" right after creation | It says "available" until attached | In-use = attached to a server |
| The snapshot is stored inside the volume | It is stored in S3, managed by AWS | It survives even if the volume is deleted |
| Deleting the file changes the snapshot | The snapshot stays as it was | It is a photo of one moment |
| Restore the restored disk needs mkfs | mkfs would erase the data | The restored disk already has the filesystem and the files |
| The mount failed because the name was different | Duplicate UUID: the copy has the same filesystem ID as the original | xfs refuses two filesystems with the same ID; -o nouuid ignores it. dmesg showed the real reason |
| S3 stores data in one AZ | S3 is regional; EBS is the one-AZ disk | Different scope |
| Snapshot restore replaces one file | A snapshot restores a whole disk | So I restore a copy, mount it on the side and copy the file out |

## What I learned
- The extra data disk survives the server; by default the root disk is deleted on termination
- A disk can only be attached to a server in the same Availability Zone
- lsblk lists block devices; df -h shows used and free space; -f shows filesystems
- File permissions: -rw-r--r-- = 644 (r = 4, w = 2, x = 1)
- The Linux names of Nitro instance disks are nvme1n1, nvme2n1, not sdf
- A failed mount: read the real reason with sudo dmesg | tail -3
- VPC and CloudTrail: VPC actions are logged under the EC2 API

## Teach-back: 1 / 3 (scenarios 3.5 / 6)
Weak spots: the full recovery steps (snapshot first, volume FROM the snapshot, no mkfs, mount, copy), S3 is regional, why mkfs would erase the data, duplicate UUID.

## Weak-topic list for CloudOps prep
- CloudTrail, Config rules and compliance (extra practice)
- EventBridge as the link from an event to SNS
- EBS snapshot recovery steps and why no mkfs on a restored disk
- S3 versioning as the answer to an overwritten object
- SCPs, trust vs permissions policy, implicit vs explicit deny

## Cleanup
- Terminated the server, deleted both volumes and the snapshot (checked: no volumes, no snapshots)

## I can explain
- The difference between S3, EBS and EFS and when to use each
- Why a snapshot is a photo stored in S3 and survives the volume
- Why I restore a copy and copy the file out instead of replacing the live disk

## Next
- Fri Nov 20: practice assessment (timed mixed questions, extra on CloudTrail, Config rules and compliance)

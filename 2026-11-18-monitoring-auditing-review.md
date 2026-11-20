# CloudOps prep day 3: monitoring and auditing review (done Nov 19)

## The problem
When something changes or breaks, I need to know which service answers which question. I used my own network break from Nov 17 as the case: find who deleted the route, check my alarms, and read a security group's history.

## What I did
| Piece | What I did | Result |
|---|---|---|
| CloudTrail Event history | Searched DeleteRoute and CreateRoute from Nov 17 | Delete at 21:35:34 PT, restore at 21:40:06 PT: route gone 4 min 32 s |
| CloudWatch alarms | Checked the 3 Lambda error alarms | saveInquiry and updateInquiry: Insufficient data; listInquiries: OK |
| CloudWatch fix | saveInquiry-errors: Missing data treatment from "missing" to "not breaching" | State changed to OK, no email (no ALARM action fired) |
| AWS Config | Read the resource timeline of web-sg | Only CloudTrail events; no configuration snapshots for the time I asked |

## Camera and photo album
| Service | Question it answers | In my lab |
|---|---|---|
| CloudWatch | How is it performing, alert me | Alarms on Lambda Errors |
| CloudTrail | WHO called which API, when, from where | DeleteRoute and CreateRoute events |
| Config | WHAT did the resource look like before and after | Only works for periods when the recorder was on |

## Reading a CloudTrail event
| Field | Tells me |
|---|---|
| userIdentity (type AssumedRole, role name, session name) | Who: the role is shared, the session name is the person |
| sessionCredentialFromConsole true, userAgent Chrome | Done from the web console, not the CLI |
| requestParameters | What exactly: the route table and the 0.0.0.0/0 route |
| eventName | Authorize... adds a rule, Revoke... removes one, RunInstances launches a server |
| eventSource | Which service API: VPC actions show as ec2.amazonaws.com |

## Breaks and wrong guesses
| What I thought | What is true | Why |
|---|---|---|
| CloudTrail keeps a before and after picture | It records the API call; Config keeps before and after | Two jobs: camera at the door vs photo album |
| The VPC console action shows vpc.amazonaws.com | It shows ec2.amazonaws.com | VPC has no API of its own, it is part of the EC2 API |
| The alarm will go to ALARM when I treat missing data as good | It goes to OK | "Not breaching" counts missing data as a good value; ALARM is the "breaching" option |
| The RunInstances call fixed the security group | The fix was RevokeSecurityGroupIngress | Revoke removes a rule, RunInstances launches a server |
| CloudTrail plus SNS alerts me when SSH opens | Nothing connects them | Need an EventBridge rule on the event, or a Config rule, then SNS |
| A period is just a range of time | It is the bucket size for one data point (for example 5 minutes) | Alarms evaluate one value per period |

## What I learned
- CloudTrail only records API calls (console, CLI and SDK all become API calls)
- Event history keeps 90 days of management events; a trail to S3 keeps them longer
- Insufficient data is not a failure: Lambda sends the Errors metric only while it runs
- Config can only show history for the time its recorder was on
- A Config rule or an EventBridge rule is the link that reacts and notifies through SNS

## Teach-back: 3 / 4 (scenarios 5.5 / 6)
Weak spots: why VPC shows as EC2 (the VPC features are part of the EC2 API), and what triggers the SNS warning (EventBridge or a Config rule).

## Weak-topic list for CloudOps prep
- SCPs (ceiling, never grant)
- Trust policy vs permissions policy
- Implicit vs explicit deny
- EventBridge as the link between an event and SNS
- Config history only exists while recording

## Cleanup
- Nothing created. Kept the alarm change (missing data = not breaching)

## I can explain
- Which of CloudWatch, CloudTrail and Config answers a given question
- How to read a CloudTrail event to find who, what, when and how
- Why an alarm shows Insufficient data and how to treat missing data

## Next
- Thu Nov 19 topic (storage and resilience review): S3, EBS and EFS, backups and restore

# Day: AWS Config

## The problem
CloudTrail tells me *who* opened port 22 on a security group. But nothing warns me that a security group is unsafe *right now*, and nothing keeps a before/after picture of how it looked.

## What I built
- Turned on AWS Config, recording only `AWS::EC2::SecurityGroup` (continuous), with the AWS Config service-linked role and a new S3 bucket for the snapshots
- Added the AWS managed rule `restricted-ssh` (checks that no security group allows SSH 22 from 0.0.0.0/0)
- All 3 security groups started Compliant (web-sg, deo-lab-default-sg, default-vpc-sg)
- Break: added SSH 22 from 0.0.0.0/0 to web-sg. The rule marked it Noncompliant within minutes
- Fix: removed the SSH rule, web-sg went back to Compliant
- Read web-sg's resource timeline and the JSON diff of the change
- Cleanup: stopped recording and deleted the rule (the S3 bucket holds a few KB and stays)

## Config in one picture: a building inspector
| Config word | Inspector version |
|---|---|
| Recorder | A camera that photographs the door every time it changes |
| Rule | The checklist: "the door must not be open to everyone" |
| Compliance | The verdict: Compliant (pass) or Noncompliant (fail) |
| Timeline | The photo album: every before and after |

## CloudWatch vs CloudTrail vs Config
| Service | Answers |
|---|---|
| CloudWatch | Is it healthy? (metrics, logs, alarms) |
| CloudTrail | Who did what, when, from where? (API calls) |
| Config | What does it look like, what did it look like before, and does it follow my rules? |

## Types of controls
| Type | What it does | AWS example |
|---|---|---|
| Detective | Finds and reports a problem after it happens | Config rules |
| Preventive | Stops it from happening | IAM policies, SCPs |
| Corrective | Fixes it automatically | Config remediation (SSM automation) |

## What I learned
- AWS resource types are written `Provider::Service::Resource`, e.g. `AWS::EC2::SecurityGroup` (security groups belong to EC2)
- Config only records from the moment it is turned on. CloudTrail is always on, so yesterday's SSH change showed CloudTrail events but no Configuration change
- Each recorded change costs about $0.003 and each rule check about $0.001, so record only the resource types you need
- A Config rule is detective: it did not stop me from opening port 22, it reported it
- The resource timeline links each Configuration change (what) to the CloudTrail event that caused it (who)
- The JSON diff works like a git diff for infrastructure: green lines added, red lines removed

## Breaks
| What I tried | What happened | Why |
|---|---|---|
| Opened SSH 22 to 0.0.0.0/0 on web-sg | restricted-ssh marked web-sg Noncompliant | The rule re-checks every time a security group changes |
| Looked for yesterday's change in the timeline | Only CloudTrail events, no Configuration change | Config's recorder was not on yet |

## I can explain
- What the recorder, a rule and compliance mean in Config
- The difference between detective and preventive controls
- Why Config, CloudTrail and CloudWatch answer different questions

## Still fuzzy
- Config as a whole felt abstract today. Revisit with the building inspector picture during CloudOps exam prep

## Next
- Thu Nov 5: Project 3, EC2 web server in deo-lab-vpc

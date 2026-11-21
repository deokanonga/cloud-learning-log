# CloudOps prep day 5: practice assessment and the "SSH opened" alert

## The problem
Before booking the exam I need to know where I am weak. I took a timed mixed practice set, then fixed my weakest topic with a hands-on build: get an email when someone opens SSH to the whole internet.

## Practice assessment
| Part | Score |
|---|---|
| Warm-up (3 questions) | 1.75 / 3 |
| Timed mixed set (12 questions) | 10 / 12 = 83% |

## What I built
| Piece | Name | Settings |
|---|---|---|
| Notification topic | deo-lab-alerts | SNS standard topic, my Gmail subscription Confirmed |
| Rule | ssh-opened-alert | EventBridge rule on the default bus |
| Event pattern | source aws.ec2, detail-type "AWS API Call via CloudTrail", eventName AuthorizeSecurityGroupIngress | Matches any "rule added" call |
| Target | deo-lab-alerts | SNS topic, EventBridge adds the publish permission itself |

## The chain
You (console) -> CloudTrail records the API call -> EventBridge rule matches the event -> SNS topic -> email.

## Break and fix
| Step | Result |
|---|---|
| Added SSH 22 from 0.0.0.0/0 on web-sg | Email arrived in about 1 minute with the raw CloudTrail event |
| Deleted the SSH rule | No email: that is RevokeSecurityGroupIngress, not in the pattern |

## Reading the alert email
| Field | Tells me |
|---|---|
| eventName AuthorizeSecurityGroupIngress | A rule was added |
| fromPort 22, toPort 22, cidrIp 0.0.0.0/0 | SSH open to the whole internet |
| userIdentity role name and session name | Who: the shared role, the session name is the person |
| sessionCredentialFromConsole true | Done in the web console, not the CLI |

## Breaks and wrong guesses
| What I thought | What is true | Why |
|---|---|---|
| CloudTrail plus SNS alerts me | Nothing connects them until I build the rule | EventBridge (or a Config rule) is the link |
| The rule only fires for port 22 | It fires for ANY added rule, even port 80 | The pattern filters on eventName, not on the port |
| Deleting the rule would email me too | No email | Delete is a different call: RevokeSecurityGroupIngress |
| S3 lives in one Availability Zone | S3 is regional, EBS is one AZ | Different scope |

## What I learned
- The link from an event to an email is an EventBridge rule with an SNS target
- A rule only reacts to events that happen AFTER it exists
- A real SSH alert would also filter on port 22 and 0.0.0.0/0
- Config rule restricted-ssh is the other way: it judges Compliant/Noncompliant (needs the recorder on)

## Weak-topic list for CloudOps prep
- Config rules and compliance (extra practice)
- EventBridge pattern filters
- EBS snapshot recovery steps and why no mkfs on a restored disk
- S3 versioning and overwritten objects
- SCPs, trust vs permissions policy, implicit vs explicit deny

## Cleanup
- Deleted the SSH rule from web-sg (back to HTTP and HTTPS only)
- EventBridge rule ssh-opened-alert: delete it or keep it (it is free, but it emails for every added rule)

## I can explain
- How to turn a CloudTrail event into an email
- Why my rule did not fire when I deleted the rule
- How to read who, what and how in the alert email

## Next
- Mon Nov 23: readiness assessment

# Day: CloudWatch and CloudTrail

## The problem
The website's inquiry Lambda got 0 invocations for a week and nobody noticed. And if someone changes a security group or redeploys a function, how would I know who did it?

## What I built
- Found a real bug through CloudWatch: the redesigned site (Cloudflare, Nov 2) used a mailto form, so saveInquiry never ran. Rewired the form to POST to the API, deployed, and verified 1 invocation, 0 errors, and the email arrived
- Listed my 3 Lambda error alarms in CloudShell (`aws cloudwatch describe-alarms --query "MetricAlarms[].AlarmName"`)
- Forced saveInquiry-errors into ALARM with `aws cloudwatch set-alarm-state` and received the SNS email
- Built the CloudWatch dashboard deokanonga-ops: Invocations, Errors and Duration for saveInquiry, plus an alarm status widget for all 3 alarms
- Found today's saveInquiry deploy (UpdateFunctionCode20150331v2) in CloudTrail Event history
- Added an SSH 22 rule to web-sg, removed it, and found both AuthorizeSecurityGroupIngress and RevokeSecurityGroupIngress in Event history
- Created a CloudTrail trail (management events, read + write, log file validation on, no SSE-KMS, no data events or Insights) that saves logs to S3

## CloudWatch vs CloudTrail
| | CloudWatch | CloudTrail |
|---|---|---|
| Answers | What is happening? Is it healthy? | Who did what, when, from where? |
| Records | Metrics, logs, alarms | API calls (every console click, CLI command, SDK call) |
| Example | saveInquiry Errors = 0, Duration 597 ms | AuthorizeSecurityGroupIngress on web-sg from 97.94.15.43 |
| Alerts | Alarms -> SNS -> email | Feeds into CloudWatch or EventBridge if you want alerts |
| Kept | Metrics up to 15 months | Event history 90 days, a trail keeps logs in S3 for as long as you want |

## Reading a CloudTrail event
| Field | Tells you |
|---|---|
| eventTime | When |
| eventName / eventSource | What, and which service (lambda.amazonaws.com, ec2.amazonaws.com) |
| userIdentity | Who. Mine is AssumedRole AccountFullAccessRole plus my sign-in session ID |
| sourceIPAddress | From where |
| userAgent | With what tool (Chrome on Mac = console, aws-cli, Boto3) |
| requestParameters | Exactly what changed (port 22, my IP /32) |
| errorCode | Empty = it worked, AccessDenied = it was blocked |

## What I learned
- No runs means no data points at all, not a 0. An Errors graph on a quiet Lambda is empty
- The Period decides the buckets. With 1-hour buckets, the current hour isn't finished, so a recent event doesn't show at 1w
- `set-alarm-state` only flips the alarm to test notifications. It creates no data, and the alarm goes back by itself at the next check
- INSUFFICIENT_DATA is the normal resting state for an Errors alarm on a function that rarely runs
- Every API call is its own CloudTrail event: adding and removing a rule are two events
- A trail's CloudTrail-Digest folder holds hourly fingerprints that prove no log file was changed or deleted
- The first trail of management events is free; a second one, data events and Insights cost money
- A strange IP plus a tool I never use (aws-cli from Linux) in CloudTrail = sign of stolen keys

## Breaks
| What I tried | What happened | Why |
|---|---|---|
| Typo `MetricAlarms[]>AlarmName` in the query | Returned null | JMESPath needs a dot: `MetricAlarms[].AlarmName` |
| Opened SSH 22 on web-sg on purpose | CloudTrail recorded who, when and from which IP | Every change is an API call, and every API call is logged |

## I can explain
- When to open CloudWatch and when to open CloudTrail
- How an alarm reaches my inbox (metric -> alarm -> SNS -> email)
- How to find who opened a port and from where
- Why an Errors graph is empty instead of 0

## Next
- Thu Nov 5: Project 3, EC2 web server in deo-lab-vpc, then attach deo-lab-web-nacl to see the NACL break live

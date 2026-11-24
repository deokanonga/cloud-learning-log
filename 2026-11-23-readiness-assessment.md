# CloudOps prep day 6: readiness assessment (practice test 2)

## The problem
Before booking the CloudOps exam I need to know if I am ready. I took a warm-up quiz on last week's topics and a timed mixed set of 12 questions, then listed my weak spots.

## Scores
| Part | Score |
|---|---|
| Warm-up quiz (5 questions) | 5 / 5 (all answers happened to be B, so I also explained two of them in words) |
| Timed mixed set (12 questions) | 9 / 12 = 75% |
My rule: book the exam after 3 different practice tests at 80% or more. Test 1 was 83%, test 2 was 75%.

## Question topics
| Topic | Result |
|---|---|
| CloudTrail trail to S3 for long retention | Right |
| Config managed rule to check encryption | Right |
| EventBridge rule on EC2 state change to SNS | Right |
| SCP limits even an admin | Right |
| S3 versioning, get the previous version | Right |
| EBS recovery: volume from snapshot, mount on the side, copy the file | Right |
| S3 is regional, EBS is one AZ | Right |
| Alarm missing data treated as not breaching | Right |
| NAT gateway route for a private subnet | Right |
| Trust policy vs permissions policy | Wrong |
| Explicit deny wins over allow | Wrong |
| Which CloudTrail field shows the person on a shared role | Wrong |

## Breaks and wrong guesses
| What I thought | What is true | Why |
|---|---|---|
| A Lambda "not authorized to assume role" needs a permissions policy fix | Fix the trust policy | Trust = WHO can become the role. Permissions = WHAT the role can do |
| An explicit deny can lose to a bucket policy or MFA | An explicit Deny always wins | Order: explicit Deny, then Allow, otherwise implicit deny |
| eventSource shows the person | The session name in userIdentity shows the person | eventSource is only which service API was called |
| The NACL outbound rule is for the visitor's IP address | It is for the visitor's port (1024-65535) | NACLs are stateless, replies need their own rule |

## What I learned
- Spot the pattern in a quiz, but answer from knowledge
- Wrong answers cluster in IAM, the topic on my weak list since Nov 16
- Decide the exam date with data: a third test on Fri Nov 27

## Weak-topic list for CloudOps prep
- Trust policy vs permissions policy
- Explicit deny vs implicit deny
- Reading CloudTrail fields (userIdentity session name)
- NACL reply ports
- Config rules and compliance (extra practice)

## Cleanup
- Nothing created

## I can explain
- How I decide when to book the exam
- Why explicit deny beats every allow
- Who can assume a role (trust policy) vs what the role can do (permissions policy)

## Next
- Tue Nov 24: IAM lab (trust vs permissions, explicit deny)
- Wed Nov 25: Config rules and CloudTrail fields
- Fri Nov 27: practice test 3, then book Dec 1 (80% or more) or Dec 8

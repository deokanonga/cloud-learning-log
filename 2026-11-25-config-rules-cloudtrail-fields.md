# CloudOps prep day 8: Config rules and CloudTrail fields

## The problem
My weak spots on the practice tests were Config rules/compliance and reading CloudTrail fields. My gallery bucket also had no versioning, which is a real gap. I used the gap as the lab.

## What I built
| Piece | Setting |
|---|---|
| Config recorder | Was off (security groups only). Re-aimed at S3 buckets, continuous, then started |
| Config rule | AWS managed rule s3-bucket-versioning-enabled (trigger: configuration changes) |
| Test buckets | Gallery bucket (no versioning) and lab bucket (versioning on) |

## Steps
| Step | What I did | Why |
|---|---|---|
| 1 | Checked Config Settings: recording was off | A rule needs recorded state to check |
| 2 | Recorded S3 buckets only, continuous | Narrow recorder keeps cost near zero |
| 3 | Added the rule | The checklist |
| 4 | Rule showed "No evaluation results" | Rule is change-triggered, nothing had changed yet |
| 5 | Added a tag to both buckets | Any recorded change takes a new photo and triggers the check |
| 6 | Verdicts: lab bucket Compliant, gallery bucket Noncompliant | Versioning was the only difference |
| 7 | Enabled versioning on the gallery bucket | The real fix |
| 8 | 4 minutes later: both Compliant | Change, new photo, checklist, verdict |
| 9 | Read the Config before/after diff: versioning Off to Enabled | Config shows WHAT changed (state) |
| 10 | Found PutBucketVersioning in CloudTrail: Status Enabled, from the console | CloudTrail shows the ACTION and WHO |
| 11 | Cleanup: deleted the rule, stopped the recorder, removed the tags, kept versioning on | Nothing left costing money |

## Breaks and wrong guesses
| What I thought | What is true | Why |
|---|---|---|
| The rule would show results right away | No results until a bucket changed | Change-triggered rules only evaluate on a configuration change |
| Only the changed setting triggers a check | Any change, even a tag, triggers a full check | A new photo is taken and the whole bucket is inspected |
| AbacStatus is bucket versioning | It is tag-based access control, unrelated | Versioning is in BucketVersioningConfiguration, status Enabled |
| The Config JSON shows the action | It shows the state; the diff shows Off to Enabled | The action (PutBucketVersioning) lives in CloudTrail |
| userIdentity type tells me the person | It only says AssumedRole | The session name at the end of userIdentity arn is the who |
| A rule works with the recorder off | It has nothing to check | Rule without recorder = inspector with no album |

## What I learned
- Recorder takes the photos, the rule is the checklist, Config shows WHAT changed, CloudTrail shows WHO did it
- Change-triggered rules wait for a change; periodic rules re-check on a schedule (1 to 24 hours)
- Config never fixes anything, it only gives a verdict
- Config diff: the Resource Timeline compares two photos side by side
- In CloudTrail: eventName is the action, requestParameters the detail, userIdentity arn session name the person
- The session name is only readable if someone chose it (my lab role session deo-test); a console login gets an id

## Cleanup
- Rule deleted, recorder stopped, tags removed
- Versioning on the gallery bucket kept ON

## I can explain
- Why a Config rule showed nothing at first
- The difference between Config (state) and CloudTrail (actions)
- Which CloudTrail field shows who

## Next
- Thu Nov 26: Thanksgiving, light review only
- Fri Nov 27: practice test 3, then book Dec 1 (80% or more) or Dec 8

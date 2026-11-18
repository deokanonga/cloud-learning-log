# CloudOps prep day 2: networking review, 3 breaks on purpose

## The problem
A public web server must stay reachable, and when it stops, I must find which layer broke. I rebuilt deo-lab-vpc from memory, launched a test server and broke three layers one by one: the NACL deny order, the NACL reply rule, and the route to the internet.

## What I built
| Piece | Name | Settings |
|---|---|---|
| VPC | deo-lab-vpc | 10.0.0.0/16, us-east-2, 4 subnets (public-a/b 10.0.1.0 and 10.0.2.0, private-a/b 10.0.11.0 and 10.0.12.0, all /24) |
| Test server | deo-lab-web2 | In public-a, Apache by user data, no key pair, web-sg; terminated at the end |
| NACL for the breaks | deo-lab-web-nacl | Attached to public-a only during the breaks |

## The 3 breaks
| Break | What I changed | What I saw | Reading |
|---|---|---|---|
| 1 | Inbound rule 90 Deny port 80 before rule 110 Allow port 80 | Page timed out | NACL rules run lowest number first; first match wins, rule 110 never runs |
| 2 | Deleted outbound rule 110 (TCP 1024-65535) | Page loaded forever, then timed out | Request got in, the reply could not leave: NACLs are stateless |
| 3 | Deleted 0.0.0.0/0 to deo-lab-igw in deo-lab-public-rt | Page timed out | No route to the IGW, so no road to the internet |

## Security group vs NACL
| | Security group | NACL |
|---|---|---|
| Level | Instance | Subnet |
| State | Stateful: remembers the request, reply allowed automatically | Stateless: request and reply each need a rule |
| Rules | Allow only | Allow and Deny, numbered, first match wins |
| No match | Implicit deny | * rule denies |

## Breaks and wrong guesses
| What I thought | What is true | Why |
|---|---|---|
| Rule 90 Deny 80 in the Outbound list would block the page | The page still loaded | Outbound rules match where the packet is going (the visitor's random port), not port 80 coming in. The deny had to be inbound |
| 1024-65535 is the server's port | It is the visitor's random port range | The reply goes TO the visitor's port; NACLs match the destination port in both directions |
| public-a stays public because the route table is still attached | A subnet is public only while its route table has 0.0.0.0/0 to the IGW | Without that route it is local-only, like a private subnet |
| Untick public-a from deo-lab-web-nacl to remove it | It jumps back to the default NACL | A subnet always has exactly one NACL |

## What I learned
- Timeout (silent drop) is different from connection refused (a server answered no)
- NACL rule order and direction matter; a deny in the wrong list does nothing
- The IGW attaches to the VPC; the route makes the subnet public
- When a public instance times out, check the route table, the NACL and the security group, in that order along the packet path

## Teach-back: 4 / 5
Weak spot: question 5. I named the right three layers but not what each check proves, and the packet-path order starts with the route table.

## Cleanup
- public-a back on default deo-lab-nacl (all 4 subnets), deo-lab-web2 terminated, no flow logs left on the VPC or public-a
- Kept: deo-lab-web-nacl (documented in the Project 3 README)

## I can explain
- Why a stateless NACL needs a reply rule and a security group does not
- Why rule 90 beats rule 110 and what an outbound deny does and does not block
- Why deleting one route turns a public subnet into a private one

## Next
- Wed Nov 18: monitoring and auditing review (CloudWatch, CloudTrail, Config)

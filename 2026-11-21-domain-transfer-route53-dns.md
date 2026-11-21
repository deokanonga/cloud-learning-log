# Domain transfer to Route 53 and moving DNS to AWS

## The problem
My domain was registered at Cloudflare, my DNS also ran on Cloudflare, and only part of my stack was on AWS. I wanted everything in one place: registrar, DNS, website, form, database and email.

## What I built
| Piece | Before | After |
|---|---|---|
| Registrar (who owns the name) | Cloudflare | Route 53 (transfer lock ON, auto-renew ON) |
| DNS (the phone book) | Cloudflare | Route 53 hosted zone |
| Website | A Cloudflare Worker | S3 + CloudFront |
| Email in | Cloudflare Email Routing | ImprovMX forwarding to Gmail |
| Email out | SES | SES (unchanged) |
| Tracker login | CloudFront address only | www address too (Cognito callback + sign-out URLs) |

## Steps
| Step | What I did | Why |
|---|---|---|
| 1 | Unlocked the domain at Cloudflare, got the authorization code | Transfers need the lock off and a code (DNSSEC was off) |
| 2 | Route 53 > Transfer in, pasted the code, kept auto-renew and privacy on | Starts the transfer; one extra year is added |
| 3 | Approved the transfer in Cloudflare | Finishes in minutes instead of waiting 5 days |
| 4 | Compared the Cloudflare DNS records with the Route 53 zone | They were NOT the same: site and email differed |
| 5 | Uploaded the current site to S3, invalidated CloudFront, compared | Otherwise visitors would have seen an old copy |
| 6 | Checked the ImprovMX aliases | Email in had to work before the switch |
| 7 | Switched the name servers to the 4 Route 53 ones (Copy name servers from a hosted zone) | Moves DNS to AWS |
| 8 | Tested site, form, email in and out | All passed, ImprovMX turned green |
| 9 | Turned the transfer lock ON | Protects against hijacking |
| 10 | Added the www address to the Cognito callback and sign-out URLs | Login works on my own domain |

## Registrar vs DNS
| Thing | Job |
|---|---|
| Registrar | Who holds the name; keeps a list of which name servers answer for it |
| DNS | The phone book: which server serves the site and the email |
The transfer moved the registrar only. DNS moved later, as its own step, after the check.

## Breaks and wrong guesses
| What I thought | What is true | Why |
|---|---|---|
| Moving the domain moves the DNS too | DNS stayed on Cloudflare | They are two separate settings |
| The Route 53 zone is a copy of Cloudflare | It had old records (CloudFront, ImprovMX) | Site and email would have changed on switch |
| We removed ImprovMX | It is the active email receiver | The MX records point to it |
| ImprovMX handles sending too | Incoming: ImprovMX. Outgoing: SES | Different jobs |
| Cognito redirects me to another page | It only allows or blocks the address on the list | Exact-match allow-list |
| Opening the apex address works with login | redirect_mismatch | Only the www address was on the list |
| Removing the sign-out URL sends me to the CloudFront page | "Invalid request" error | The logout address was not allowed. Put it back and logout worked |

## What I learned
- Registrar and DNS are separate; check both zones record by record before switching name servers
- Name server changes spread slowly; both sides should give the same result during the switch
- ImprovMX = email in, SES = email out
- Cognito: callback URLs = login, sign-out URLs = logout, both exact-match
- Turn the transfer lock on after a transfer

## Cleanup
- Cloudflare zone, Worker and Email Routing are kept as a fallback for a few days, then removed
- AdSense was not deployed (ads.txt gave 404)

## I can explain
- The difference between registrar and DNS
- Why I compared both DNS zones before moving
- How the Cognito allow-lists protect the login

## Next
- Remove Cloudflare leftovers in 3 to 7 days
- Mon Nov 23: readiness assessment

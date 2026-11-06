# Project 3: EC2 web server in my own VPC

## The problem
A company needs a website on a server. It must be reachable on the web, with no SSH door open to the world, no access keys on the server, and an alert if the server gets overloaded.

## What I built
| Piece | Name | Settings |
|---|---|---|
| IAM role (instance profile) | deo-lab-ec2-role | Trusted by EC2, policy AmazonSSMManagedInstanceCore |
| EC2 instance | deo-lab-web | Amazon Linux 2023, t3.micro, no key pair, deo-lab-vpc, subnet deo-lab-public-a, public IP, security group web-sg (80/443 in, no 22) |
| Web page | (user data) | Script installs Apache (httpd) and writes index.html at first boot |
| Remote access | Session Manager | No SSH, no port 22, no keys |
| CPU alarm | deo-lab-web-cpu-high | CPUUtilization > 70% for 1 x 5 min, SNS topic deo-lab-alerts -> Gmail |
| NACL break + fix | deo-lab-web-nacl | Attached to public-a, broke it, fixed it with 3 rules |

User data script:
```
#!/bin/bash
dnf install -y httpd
echo "<h1>Hello from deo-lab-web</h1><p>Project 3: EC2 in deo-lab-vpc</p>" > /var/www/html/index.html
systemctl enable --now httpd
```

## How traffic reaches the page
Internet -> deo-lab-igw -> deo-lab-public-rt (0.0.0.0/0 -> IGW) -> deo-lab-public-a -> NACL -> web-sg (port 80) -> Apache

## Session Manager: two connections
| Connection | Path | Crosses the NACL? |
|---|---|---|
| 1 | Me (browser) <-> AWS Systems Manager on 443 | No, outside my VPC |
| 2 | SSM agent on the server <-> AWS Systems Manager | Yes: OUT 443, replies IN on 1024-65535 |

AWS joins the two like a call centre operator. The agent always calls out first, so nothing needs an inbound door. Access is controlled by IAM: the server's role lets the agent register, and my own sign-in role lets me start a session.

## Inside the server
- `whoami` -> ssm-user (Session Manager's own user)
- `systemctl status httpd` -> active (running), enabled, total requests 3 (my own visits)
- Servers are headless: no desktop, only the command line
- Installed git with `sudo dnf install -y git` and wrote a file with a heredoc (`cat <<EOF > file.txt`)

## The NACL fix
| Direction | Rule # | Port | Why |
|---|---|---|---|
| Inbound | 100 | 443 | (already there) HTTPS in |
| Inbound | 110 | 80 | Visitors reach the page |
| Inbound | 120 | 1024-65535 | AWS's replies reach the SSM agent |
| Outbound | 100 | 443 | (already there) SSM agent calls AWS |
| Outbound | 110 | 1024-65535 | The page's replies reach the visitor's random port |

## What I learned
- On AWS, Session Manager is the default way in. Port 22 needs a reason, and then only from a known IP range
- To pick IAM policies, ask "what will this server talk to?" The service docs list the policy (e.g. Session Manager prerequisites)
- User data runs once, as root, at first boot. This is called bootstrapping
- Anything installed by hand dies with the server. Put it in user data, a custom AMI or Terraform so every new server comes out the same
- The CloudWatch metrics list only shows the region you are in (I was in N. Virginia, my server was in Ohio)
- The alarm checks its own 5-minute window. The graph's newest bucket shows up late
- A NACL rule number is only the order. 110 is not a port (even though port 110 exists: POP3)
- A NACL rule checks the To port. Servers listen on fixed ports (80, 443), clients use random ports (1024-65535)
- NACLs are stateless, so every reply needs its own rule. Security groups are stateful and handle replies themselves
- A subnet always has exactly one NACL. If you remove a custom one, it goes back to the default
- Open network path does not mean a service is running: 443 was allowed but nothing listened on it

## Breaks
| What I tried | What happened | Why |
|---|---|---|
| Launch defaults | Default VPC and a new SG with SSH from 0.0.0.0/0 | Changed to deo-lab-vpc and web-sg |
| EC2 Instance Connect | "Port 22 (SSH) is not authorized" | web-sg has no port 22 rule |
| Two `yes > /dev/null &` processes | CPU 90%, alarm went OK -> ALARM, email arrived | Pure CPU work on both vCPUs |
| Attached deo-lab-web-nacl | Website timed out, Session Manager lost connection | No inbound 80, no reply ports in or out |
| `https://<IP>` after the fix | Failed | Apache only listens on 80, no certificate |

## Cleanup
- Terminated deo-lab-web (disk deleted with it)
- Deleted the deo-lab-web-cpu-high alarm
- Put deo-lab-public-a back on the default NACL
- Kept for free: deo-lab-ec2-role, web-sg, deo-lab-web-nacl (fixed rules), SNS topic deo-lab-alerts

## I can explain
- How Session Manager gets in with no port 22 and no keys
- Why a NACL needs the 1024-65535 range and a security group doesn't
- Why hand-installed software should live in user data, an AMI or Terraform

## Next
- Nov 9-13: storage, backups and recovery (point-in-time recovery for the Inquiries table)

# VPC Endpoints — Cheat Sheet

Lesson notes for reviewing later. The project write-up is in [README.md](README.md). This is the **final project of the NextWork networking series**.

The first half of this project (EC2 Instance Connect, access keys, `aws configure`, `aws s3` commands) repeats the previous project, so those notes are in the [Access-S3-from-a-VPC cheat sheet](https://github.com/jasahorro/Access-S3-from-a-VPC/blob/main/CHEATSHEET.md). This sheet focuses on what's new.

## Contents

1. [What I built](#what-i-built)
2. [Key terms](#key-terms)
3. [The problem: S3 lives outside your VPC](#the-problem-s3-lives-outside-your-vpc)
4. [VPC endpoints: gateway vs interface](#vpc-endpoints-gateway-vs-interface)
5. [Setting up the VPC and instance](#setting-up-the-vpc-and-instance)
6. [Bucket policy: only allow the endpoint](#bucket-policy-only-allow-the-endpoint)
7. [The mystery: why EC2 was denied too](#the-mystery-why-ec2-was-denied-too)
8. [Endpoint policies](#endpoint-policies)
9. [Bucket policy vs endpoint policy](#bucket-policy-vs-endpoint-policy)
10. [Cleanup (harder than usual!)](#cleanup-harder-than-usual)
11. [Self-quiz](#self-quiz)

---

## What I built

- 🌐 **A VPC** (`NextWork-vpc`, `10.0.0.0/16`) with one public subnet in one Availability Zone.
- 💻 **An EC2 instance** in the public subnet, reached with EC2 Instance Connect, using access keys and the AWS CLI to work with S3.
- 🚪 **An S3 gateway endpoint** that gives the VPC a private route to S3.
- 🔐 **A bucket policy** that denies every request unless it comes through that endpoint.
- 🛣️ **A route table update** that sends S3-bound traffic from the public subnet to the endpoint.
- 🧪 **An endpoint policy test**: switching it from Allow to Deny cut off access straight away.

```
                    ┌──────────────────── NextWork-vpc (10.0.0.0/16) ───────────────────┐
                    │  public subnet                                                    │
                    │  ┌──────────────┐     route table:                                │
  Me ──Instance ───►│  │ EC2 instance │     0.0.0.0/0    → internet gateway           │
       Connect      │  │ (AWS CLI)    │     pl-xxxx (S3) → vpce-xxxx  ◄── added in this │
                    │  └──────┬───────┘                                     project     │
                    └─────────┼─────────────────────────────────────────────────────────┘
                              │ private route (stays on the AWS network)
                              ▼
                    ┌──────────────────────┐      ┌───────────────────────────────┐
                    │ S3 gateway endpoint  │ ───► │ S3 bucket                     │
                    │ (endpoint policy)    │      │ bucket policy: deny unless    │
                    └──────────────────────┘      │ request came via vpce-xxxx    │
                                                  └───────────────────────────────┘
  Console (public internet) ──────────────────────────────► ✖ Access Denied
```

## Key terms

| Term | Plain meaning | Analogy |
|---|---|---|
| **VPC endpoint** | A private connection from your VPC to an AWS service, without going over the public internet | A private door from your building straight into the service's building |
| **Internet gateway** | The VPC's connection to the public internet | The building's front door to the street |
| **Gateway endpoint** | An endpoint type for **S3 and DynamoDB only**. Works by adding a route to your route table. Free | A private tunnel with a sign on the road pointing to it |
| **Interface endpoint** | An endpoint type for many AWS services. Puts a network interface with a private IP in your subnet. Costs per hour | A private service desk set up inside your own building |
| **Route table** | Rules that decide where traffic from a subnet goes | The GPS for your subnet's traffic |
| **Prefix list** (`pl-…`) | A named set of IP ranges, here all of S3's addresses in the region | A contact group: "send everything for S3 here" |
| **Bucket policy** | A JSON policy on an S3 bucket saying who can do what with it | The rules posted on a storage locker's door |
| **Endpoint policy** | A JSON policy on a VPC endpoint saying what traffic may pass through it | The security guard standing in the private tunnel |
| **Explicit deny** | A `"Deny"` statement. It always beats any `"Allow"` | A "No entry" sign that overrules every guest pass |

## The problem: S3 lives outside your VPC

- **Not all AWS resources live inside your VPC.** EC2 instances do, but S3 (and services like DynamoDB) sit **outside** it so they're reachable from anywhere.
- Without an endpoint, the instance reaches S3 **through the internet gateway, over the public internet**.
- **With a VPC endpoint**, traffic goes privately and stays on the AWS network.

**Why the private route is better:**

| Benefit | Why it matters |
|---|---|
| **No public internet path** | Less exposure to outside threats |
| **Lock the bucket to your VPC** | A bucket policy can say "only accept requests through this endpoint" (this project) |
| **Works from private subnets** | Private instances can reach S3 without a NAT gateway |
| **Lower cost** | You avoid NAT gateway data-processing charges, and gateway endpoints themselves are free |

> **Correction to the lesson:** it says attackers could grab your keys by breaking into the traffic. In practice, the AWS CLI already talks to S3 over **HTTPS (encrypted)**, and your secret key is never sent. It's only used to sign each request. The real security wins are the ones in the table: no public path, and being able to lock the bucket down to your VPC.

## VPC endpoints: gateway vs interface

| | Gateway endpoint (*used in this project*) | Interface endpoint |
|---|---|---|
| Services | **S3 and DynamoDB only** | Most AWS services (and S3 too) |
| How it works | Adds a **route** (prefix list → `vpce-…`) to the route tables you pick | Creates a **network interface (ENI)** with a private IP in your subnet |
| Security controls | Endpoint policy + route tables | Endpoint policy + **security groups** + private DNS |
| Cost | **Free** | Charged per hour + per GB |
| Reachable from on-premises or peered VPCs? | No | Yes |

- **Don't mix up "Gateway" with internet gateway or NAT gateway.** A gateway endpoint is a private route to S3/DynamoDB. It doesn't connect to the internet.
- **History:** gateway endpoints came first but offered few settings. **Interface endpoints** (built on **AWS PrivateLink**) were added later for more services and more security controls.

## Setting up the VPC and instance

**VPC → Create VPC → "VPC and more":**

| Setting | Choice | Why |
|---|---|---|
| Name tag auto-generation | `NextWork` | Names everything `NextWork-…` |
| IPv4 CIDR | `10.0.0.0/16` (default) | 65,536 private addresses |
| IPv6 CIDR | No IPv6 | Not needed |
| Tenancy | Default | Dedicated hardware costs more |
| Availability Zones | 1 | Keep it simple |
| Public / private subnets | 1 / 0 | Only need one public subnet for Instance Connect |
| NAT gateways | **None** | NAT gateways cost money 💲 |
| VPC endpoints | **None** | We set one up manually later to learn how. (This box could have added an S3 gateway endpoint for you.) |
| DNS options | Leave checked | |

**EC2 → Launch instance:**
- Name `Instance - NextWork VPC Project`, AMI **Amazon Linux 2023**, type **t2.micro**.
- **Proceed without a key pair**. That's fine because EC2 Instance Connect doesn't need one.
- Network: `NextWork-vpc`, the **public subnet**, **Auto-assign public IP: Enable**. Instance Connect needs a public subnet *and* a public IPv4 address.
- New security group `SG - NextWork VPC Project`. Its default inbound **SSH** rule is all Instance Connect needs. No ICMP rule this time, because there are no ping tests.

Then, as in the last project: create an access key in IAM, run `aws configure`, create the bucket, and test with `aws s3 ls`, `touch`, `aws s3 cp` and `aws s3 ls s3://<bucket>`.

## Bucket policy: only allow the endpoint

The policy **denies everything to everyone, unless the request came through my endpoint**:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:*",
      "Resource": [
        "arn:aws:s3:::nextwork-vpc-project-jasahorro",
        "arn:aws:s3:::nextwork-vpc-project-jasahorro/*"
      ],
      "Condition": {
        "StringNotEquals": {
          "aws:sourceVpce": "vpce-xxxxxxxxxxxxxxxxx"
        }
      }
    }
  ]
}
```

| Part | Meaning |
|---|---|
| `"Effect": "Deny"` | This is a block, and an explicit deny beats any allow |
| `"Principal": "*"` | Applies to **everyone**, even me and the root user |
| `"Action": "s3:*"` | Every S3 action (list, read, upload, delete...) |
| `Resource` (two lines) | The bucket itself **and** every object in it (`/*`) |
| `StringNotEquals` + `aws:sourceVpce` | ...**unless** the request came through this endpoint ID |

**Right after saving**, the bucket page in the console showed **Access Denied** everywhere. The console reaches S3 **over the public internet**, not through the endpoint, so the policy blocked me too. That shows the policy works.

## The mystery: why EC2 was denied too

- The endpoint existed, but `aws s3 ls s3://<bucket>` **from the EC2 instance was denied** as well. 🤔
- **The answer was in the route table.** Creating a gateway endpoint does nothing until a route table **uses** it. The public subnet's route table still sent S3 traffic to the **internet gateway**. So the request went out over the internet, arrived without the endpoint ID, and the bucket policy denied it.
- **The fix:** VPC → **Endpoints** → select the endpoint → **Manage route tables** → tick the **public subnet's route table** → Save.
- That added a route: **destination = S3's prefix list (`pl-…`), target = the endpoint (`vpce-…`)**. After that, the same command listed the bucket's objects. The request now **met the policy's condition**, so it wasn't bypassing anything.

> **The rule to remember:** a gateway endpoint only works for subnets whose **route table** points to it. If you see "endpoint exists but still Access Denied", check the route table first.

## Endpoint policies

- An **endpoint policy** is attached to the endpoint and controls what traffic is allowed **through it**.
- The default is **full access** (Allow everything).
- **Test:** I changed `"Effect": "Allow"` to `"Deny"`, and the next `aws s3` command from EC2 returned **Access Denied** right away.

**Why use one:**
- 🚨 **Kill switch:** if you suspect someone is misusing the endpoint, switch it to Deny to close it instantly.
- 🎯 **Fine-grained access:** for example, allow only `s3:GetObject` and `s3:ListBucket` (read-only), so uploads through the endpoint are blocked. Or only allow your own buckets, so nobody can copy data out to someone else's bucket.

## Bucket policy vs endpoint policy

| | Bucket policy | Endpoint policy |
|---|---|---|
| Attached to | The **S3 bucket** | The **VPC endpoint** |
| Protects | Who can reach **this bucket**, from anywhere | What can travel **through this endpoint**, to any service resource |
| Question it answers | "Which requests may touch my bucket?" | "What is my VPC allowed to do over this private route?" |
| This project | Deny everything not coming via `vpce-…` | Switched Allow → Deny to block everything |

A request has to pass **both**: the endpoint policy on the way through, and the bucket policy at the bucket, plus the IAM user's own permissions.

## Cleanup (harder than usual!)

The console **can't delete the bucket**, because the bucket policy blocks it. Do it from the EC2 instance through the endpoint:

1. **Set the endpoint policy back to Allow first.** Otherwise the EC2 commands below get Access Denied too.
2. Empty the bucket: `aws s3 rm s3://nextwork-vpc-project-jasahorro --recursive`
   - `rm` = remove. `--recursive` = everything inside, including "subfolders".
   - `aws s3 rb` **fails on a bucket that isn't empty**, so empty it first. (`aws s3 rb s3://<bucket> --force` does both in one go.)
3. Delete the bucket: `aws s3 rb s3://nextwork-vpc-project-jasahorro` (`rb` = remove bucket).
4. Terminate the EC2 instance.
5. Delete the VPC endpoint, then the VPC (which also removes its subnets, route tables and internet gateway).
6. **Delete the access key in IAM**, and delete the downloaded `accessKeys.csv`.

> **Locked out with no EC2 left?** The **AWS account root user** can always delete a bucket policy, even one that denies everyone. Sign in as root, delete the policy, then delete the bucket in the console.

---

## Self-quiz

<details><summary>1. Why does EC2-to-S3 traffic go over the public internet by default?</summary>

S3 lives **outside** your VPC. Without an endpoint, the only route to it is through the internet gateway.
</details>

<details><summary>2. What is a VPC endpoint, in one sentence?</summary>

A private connection from your VPC to an AWS service, so the traffic stays on the AWS network instead of the public internet.
</details>

<details><summary>3. Which two services can use a gateway endpoint?</summary>

**Amazon S3** and **Amazon DynamoDB**.
</details>

<details><summary>4. How does a gateway endpoint actually send traffic to S3?</summary>

It adds a **route** to the route tables you choose: destination = S3's **prefix list** (`pl-…`), target = the endpoint (`vpce-…`).
</details>

<details><summary>5. Name two differences between gateway and interface endpoints.</summary>

Any two of these:
- Gateway endpoints are **free** and only for S3/DynamoDB. Interface endpoints cost money and cover most services.
- Gateway endpoints use a **route**. Interface endpoints use a **network interface with a private IP** and can have security groups.
- Interface endpoints can be reached from on-premises networks and peered VPCs.
</details>

<details><summary>6. What did the bucket policy do?</summary>

It **denied all S3 actions to everyone**, unless the request came through my VPC endpoint (`aws:sourceVpce`).
</details>

<details><summary>7. Why did the console show Access Denied right after saving the bucket policy?</summary>

The console reaches S3 over the **public internet**, not through the endpoint, so the deny applied to me too.
</details>

<details><summary>8. Why are there two `Resource` lines (`bucket` and `bucket/*`)?</summary>

The first covers the **bucket itself** (for actions like listing it). The `/*` covers **every object inside** it.
</details>

<details><summary>9. The endpoint exists, but EC2 still gets Access Denied. What's the most likely cause?</summary>

The subnet's **route table doesn't have a route to the endpoint**, so traffic still goes over the internet, without the endpoint ID the bucket policy requires.
</details>

<details><summary>10. How did you fix it?</summary>

VPC → Endpoints → select the endpoint → **Manage route tables** → tick the public subnet's route table → Save.
</details>

<details><summary>11. What's an endpoint policy, and what happened when you switched it to Deny?</summary>

A policy on the endpoint that controls what traffic may pass through it. Switching it to Deny blocked all access through the endpoint, so the EC2 commands got Access Denied immediately.
</details>

<details><summary>12. Give two reasons to use an endpoint policy.</summary>

- As a **kill switch** to close the endpoint instantly if it's being misused.
- For **fine-grained access**, like read-only (no uploads), or only your own buckets.
</details>

<details><summary>13. What's the difference between a bucket policy and an endpoint policy?</summary>

A bucket policy sits on the **bucket** and controls who can reach it from anywhere. An endpoint policy sits on the **endpoint** and controls what can travel through it. A request must pass both.
</details>

<details><summary>14. Why couldn't you delete the bucket from the console?</summary>

The bucket policy denies every request that doesn't come through the endpoint, and the console isn't using the endpoint.
</details>

<details><summary>15. Why did `aws s3 rb` fail the first time, and what fixed it?</summary>

You can't delete a bucket that still has objects in it. `aws s3 rm s3://<bucket> --recursive` emptied it first. (`aws s3 rb --force` does both.)
</details>

<details><summary>16. What must you do before emptying the bucket from EC2, if the endpoint policy is still set to Deny?</summary>

Switch the endpoint policy back to **Allow**. Otherwise your EC2 commands are blocked too.
</details>

<details><summary>17. If you lock yourself out of a bucket with a policy, who can still fix it?</summary>

The **AWS account root user**. It can always delete the bucket policy.
</details>

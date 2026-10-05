# EC2 Networking & Connectivity

## EC2 User Data versus Instance Metadata

**User Data** tells an EC2 instance what to do during launch/boot. Typical labs use it to update packages, install Apache, start services and generate files.

**Instance Metadata** lets the instance obtain information about itself.

The EC2 Instance Metadata Service uses the link-local address:

```text
169.254.169.254
```

IMDSv2 uses a token before metadata requests.

Mental association:

> User Data = instructions for the machine.  
> Metadata = information about the machine.

## Apache and heredoc

A common lab pattern:

```bash
cat <<EOF > /var/www/html/index.html
...
EOF
```

means: take the text until the delimiter `EOF` and write it to the file. Apache commonly serves web content from `/var/www/html`.

## EC2 Status Checks

**System Status Check** concerns AWS underlying infrastructure.

> Building okay?

**Instance Status Check** concerns the guest instance/OS/configuration.

> Apartment okay?

`2/2 checks passed` means both are healthy.

## ENI, ENA and EFA

```text
ENI → connects
ENA → accelerates
EFA → coordinates tightly coupled clusters
```

- **ENI**: virtual network interface.
- **ENA**: enhanced networking/high performance.
- **EFA**: very low-latency/high-throughput communication for tightly coupled HPC/ML workloads.

## Placement Groups

**Cluster** — instances close together; optimize low latency/high throughput.

> Cluster = together = performance.

**Spread** — separates individual instances to reduce correlated failure.

> Spread = separate instances.

**Partition** — separates groups of instances into hardware partitions.

> Partition = separate groups.

## Connectivity methods

**SSH** — traditional Linux terminal access, commonly port 22 plus key and network path.

**RDP** — traditional Windows GUI access, commonly port 3389.

**EC2 Instance Connect** — IAM-assisted SSH using temporary keys.

**EC2 Instance Connect Endpoint** — provides a path for SSH to private instances without making the instance public.

> EIC Endpoint = “I want SSH, but my EC2 is private.”

**Systems Manager Session Manager** — administrative sessions without requiring an inbound SSH path.

> Session Manager = “administer without opening inbound SSH.”

## Private EC2 via EC2 Instance Connect Endpoint

```text
User
  ↓
EC2 Instance Connect
  ↓
EC2 Instance Connect Endpoint
  ↓
Private VPC networking
  ↓
Security controls
  ↓
Private IP / ENI
  ↓
EC2
  ↓
SSH
```

## Linux network view

`ifconfig` can show interface status, private IPv4, netmask, MAC address, RX/TX statistics and loopback.

Modern alternatives:

```bash
ip addr
ip link
ip route
```

Important:

> The guest OS does not show AWS Route Tables, NACLs or Security Groups through ifconfig. Those controls exist in AWS networking outside the guest OS.

## SSH private-key permissions

If an SSH private key is too broadly readable, SSH may reject it as an unprotected private key.

A common lab correction:

```bash
chmod 400 ssh-lab.pem
```

Mental rule:

> Private key means private.

## AWS CLI instance launch

A launch command can be interpreted by layer:

```text
AMI             = WHAT system/image?
Subnet          = WHERE does it live?
Security Group  = WHAT network traffic is permitted?
ENI / IP        = HOW is it connected/identified?
IAM Role        = WHAT AWS actions may it perform?
EC2             = THE machine
```

This is AWS-Onion in practice: launching “one EC2” actually connects several architectural layers.

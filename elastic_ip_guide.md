# AWS Elastic IP (EIP) Guide

## Overview

Elastic IPs provide a static public IP address for EC2 instances, ensuring the same IP address is maintained even after instance stop/start cycles. This is essential for maintaining consistent DNS records and avoiding connection issues.

## Why Use Elastic IPs?

- **Static IP Address**: Maintains the same public IP across instance restarts
- **DNS Stability**: No need to update DNS records when restarting instances
- **Application Continuity**: External services can reliably connect to your instance
- **Cost Effective**: Free when associated with a running instance

## Method 1: AWS CLI Commands

### Allocate and Associate Elastic IP

```bash
# Allocate an Elastic IP
aws ec2 allocate-address --domain vpc --region ap-southeast-2

# Example output:
# {
#     "PublicIp": "13.54.123.45", 
#     "AllocationId": "eipalloc-12345678",
#     "Domain": "vpc"
# }

# Associate with your instance
aws ec2 associate-address \
  --instance-id i-1234567890abcdef0 \
  --allocation-id eipalloc-12345678 \
  --region ap-southeast-2
```

### Quick Setup Script

```bash
#!/bin/bash
# Quick EIP setup for existing instance

# 1. Allocate EIP
EIP_ALLOC=$(aws ec2 allocate-address --domain vpc --region ap-southeast-2 --query 'AllocationId' --output text)
echo "Allocated EIP: $EIP_ALLOC"

# 2. Get your instance ID
INSTANCE_ID="i-1234567890abcdef0"  # Replace with your instance ID

# 3. Associate EIP
aws ec2 associate-address \
  --instance-id $INSTANCE_ID \
  --allocation-id $EIP_ALLOC \
  --region ap-southeast-2

# 4. Get the public IP
PUBLIC_IP=$(aws ec2 describe-addresses --allocation-ids $EIP_ALLOC --query 'Addresses[0].PublicIp' --output text)
echo "Your static public IP: $PUBLIC_IP"
```

## Method 2: AWS Console

1. Go to **EC2 Console** → **Elastic IPs**
2. Click **Allocate Elastic IP address**
3. Select **Amazon's pool of IPv4 addresses**
4. Click **Allocate**
5. Select the allocated IP → **Actions** → **Associate Elastic IP address**
6. Choose your instance → **Associate**

## Method 3: Terraform Configuration

### Basic EIP Setup

```hcl
# Allocate Elastic IP
resource "aws_eip" "instance_eip" {
  domain = "vpc"
  
  tags = {
    Name = "my-instance-elastic-ip"
  }
}

# Associate EIP with instance
resource "aws_eip_association" "instance_eip_assoc" {
  instance_id   = aws_instance.my_instance.id
  allocation_id = aws_eip.instance_eip.id
}

# Your EC2 instance
resource "aws_instance" "my_instance" {
  ami           = "ami-0c02fb55956c7d316"  # Amazon Linux 2023
  instance_type = "t3.medium"
  
  vpc_security_group_ids = [aws_security_group.instance_sg.id]
  subnet_id             = aws_subnet.public.id
  
  tags = {
    Name = "my-instance"
  }
}

# Output the static IP
output "instance_public_ip" {
  description = "Static public IP for instance"
  value       = aws_eip.instance_eip.public_ip
}
```

### Advanced: EIP with Elastic Network Interface (ENI)

```hcl
# Create ENI
resource "aws_network_interface" "instance_eni" {
  subnet_id       = aws_subnet.public.id
  security_groups = [aws_security_group.instance_sg.id]
  
  tags = {
    Name = "instance-eni"
  }
}

# Attach EIP to ENI
resource "aws_eip" "instance_eip" {
  network_interface         = aws_network_interface.instance_eni.id
  domain                   = "vpc"
  associate_with_private_ip = aws_network_interface.instance_eni.private_ip
  
  tags = {
    Name = "instance-elastic-ip"
  }
}

# Attach ENI to instance
resource "aws_network_interface_attachment" "instance_eni_attachment" {
  instance_id          = aws_instance.my_instance.id
  network_interface_id = aws_network_interface.instance_eni.id
  device_index         = 1
}
```

## Method 4: Auto-Association Script

Create a startup script that automatically associates EIP on instance boot:

```bash
#!/bin/bash
# associate-eip.sh - Add to user data or startup scripts

ALLOCATION_ID="eipalloc-12345678"  # Your EIP allocation ID
INSTANCE_ID=$(curl -s http://169.254.169.254/latest/meta-data/instance-id)
REGION="ap-southeast-2"

echo "Associating EIP $ALLOCATION_ID with instance $INSTANCE_ID"

aws ec2 associate-address \
  --instance-id $INSTANCE_ID \
  --allocation-id $ALLOCATION_ID \
  --region $REGION \
  --allow-reassociation

echo "EIP association complete"
```

## Cost Structure

### Pricing Details

| Scenario | Cost |
|----------|------|
| **Associated with running instance** | **FREE** |
| **Unassociated (allocated but not attached)** | **$0.005/hour** (~$3.60/month) |
| **Associated with stopped instance** | **$0.005/hour** (~$3.60/month) |

### Cost Optimization Tips

- Always associate EIP immediately after allocation
- Release EIP if no longer needed: `aws ec2 release-address --allocation-id eipalloc-12345678`
- Use Application Load Balancer for production web applications instead of EIP
- Monitor unused EIPs in AWS Cost Explorer

## DNS Configuration

### Update DNS Records

```bash
# Route 53 example
aws route53 change-resource-record-sets \
  --hosted-zone-id Z1234567890ABC \
  --change-batch '{
    "Changes": [{
      "Action": "UPSERT",
      "ResourceRecordSet": {
        "Name": "myapp.example.com",
        "Type": "A",
        "TTL": 300,
        "ResourceRecords": [{"Value": "YOUR_ELASTIC_IP"}]
      }
    }]
  }'
```

### Manual DNS Update

For external DNS providers, update A record:

- **Record Type**: A
- **Name**: myapp.example.com
- **Value**: Your Elastic IP
- **TTL**: 300 (5 minutes)

## Management Commands

### List All Elastic IPs

```bash
# List all EIPs in your account
aws ec2 describe-addresses --region ap-southeast-2

# List unassociated EIPs (costing money)
aws ec2 describe-addresses \
  --region ap-southeast-2 \
  --query 'Addresses[?!InstanceId]'
```

### Check EIP Status

```bash
# Check specific EIP
aws ec2 describe-addresses \
  --allocation-ids eipalloc-12345678 \
  --region ap-southeast-2

# Check which instance has the EIP
aws ec2 describe-addresses \
  --region ap-southeast-2 \
  --query 'Addresses[?PublicIp==`13.54.123.45`]'
```

### Disassociate EIP

```bash
# Disassociate EIP from instance
aws ec2 disassociate-address \
  --association-id eipassoc-12345678 \
  --region ap-southeast-2

# Release EIP (stops billing)
aws ec2 release-address \
  --allocation-id eipalloc-12345678 \
  --region ap-southeast-2
```

## Alternative Solutions

### Application Load Balancer (ALB)

For web applications, consider using ALB instead of EIP:

**Advantages:**
- Static DNS name (no IP changes needed)
- Built-in health checks
- SSL termination
- Multiple availability zones
- Auto-scaling support

**When to use ALB vs EIP:**
- **EIP**: Simple applications, single instance, SSH access needed
- **ALB**: Web applications, multiple instances, production workloads

### Elastic Network Interface (ENI)

For more complex networking requirements:

**Use cases:**
- Multiple network interfaces
- License binding to MAC address
- Network-level redundancy
- Complex routing requirements

## Troubleshooting

### Common Issues

| Issue | Solution |
|-------|----------|
| **EIP not associating** | Check instance is in VPC, verify allocation ID |
| **Connection still fails** | Verify security groups allow traffic |
| **Billing for unused EIP** | Associate with instance or release |
| **DNS not resolving** | Check TTL settings, update DNS records |

### Verification Commands

```bash
# Test EIP association
aws ec2 describe-addresses --region ap-southeast-2

# Test connectivity
curl -I http://YOUR_ELASTIC_IP:8088

# Check DNS resolution
dig myapp.example.com
nslookup myapp.example.com

# Check from instance
curl -s http://169.254.169.254/latest/meta-data/public-ipv4
```

## Best Practices

### Security
- Always use security groups to restrict access
- Don't rely on IP-based security alone
- Regularly audit EIP usage

### Operations
- Tag all EIPs with purpose and owner
- Document EIP assignments
- Monitor for unused EIPs
- Use Infrastructure as Code (Terraform) for reproducible deployments

### Cost Management
- Set up billing alerts for EIP charges
- Regularly review unassociated EIPs
- Consider ALB for production web applications
- Release EIPs when decommissioning resources

## Summary

Elastic IPs provide a simple and cost-effective way to maintain static public IP addresses for EC2 instances. They're free when associated with running instances and essential for maintaining stable DNS records and external connectivity.

**Quick Decision Guide:**
- **Single instance, simple setup**: Use Elastic IP
- **Web application, production**: Consider Application Load Balancer
- **Complex networking**: Use ENI with EIP
- **Development/testing**: Regular public IPs may suffice

Remember to always associate EIPs immediately after allocation to avoid unnecessary charges!
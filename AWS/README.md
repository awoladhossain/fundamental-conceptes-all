# ☁️ AWS (Amazon Web Services) Learning Notes

## 📂 Topics Covered

| File | Description | Status |
|------|-------------|--------|
| [Chapter 01](chapter01.md) | AWS Introduction & Basics | ✅ Done |
| [EC2](EC2.md) | Elastic Compute Cloud | ✅ Done |
| [EBS](EBS.md) | Elastic Block Store | ✅ Done |
| [ELB](ELB.md) | Elastic Load Balancer | ✅ Done |

## 🎯 Quick Reference

### EC2 Instance Types
- **t2.micro** - Free tier eligible
- **t3.medium** - General purpose
- **c5.large** - Compute optimized
- **r5.large** - Memory optimized

### Common AWS CLI Commands
```bash
# List EC2 instances
aws ec2 describe-instances

# Start instance
aws ec2 start-instances --instance-ids i-xxxxx

# Stop instance
aws ec2 stop-instances --instance-ids i-xxxxx
```

## 📚 Resources
- [AWS Official Docs](https://docs.aws.amazon.com/)
- [AWS Free Tier](https://aws.amazon.com/free/)

---
*Last Updated: March 2026*

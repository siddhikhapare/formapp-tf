
## Folder Structure

```
.
├── main.tf                 # Root module: wires all child modules together
├── variables.tf            # Root input variables
├── outputs.tf              # Root outputs
├── provider.tf             # Default AWS provider + us-east-1 alias (CloudFront ACM)
├── backend.tf              # Remote state (S3) configuration
├── terraform.tfvars        # Environment values (do NOT commit secrets)
│
├── vpc/                    # VPC, public / private-app / private-db subnets, IGW, NAT, routes
├── security-group/         # Bastion, external ALB, internal ALB, web, app, RDS SGs
├── bastion-host/           # Bastion EC2 in public subnet (SSH entry point)
├── LB/                     # External ALB (public) + Internal ALB (private) + target groups
├── rds/                    # RDS subnet group + DB instance
├── ami/                    # Builds base instances and bakes custom Web / App AMIs, IAM instance profile
├── web-asg/                # Web launch template + ASG + alarms/notifications
├── app-asg/                # App launch template + ASG (injects DB config) + alarms
└── cdn/                    # CloudFront distribution + Route 53 records
```

### Module dependency graph
vpc
  → security_group
      → rds        (parallel with...)
      → alb + bastion   (parallel, both only need security_group + vpc)
          → ami    (needs alb.internal_alb_dns_name)
              → web_asg   (needs ami + alb)
              → app_asg   (needs ami + alb + rds)
                  → cdn   (needs alb — actually just needs alb, not app_asg)


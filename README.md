# AWS Auto Scaling Lab with Apache Web Server

This CloudFormation template creates a complete AWS infrastructure setup to demonstrate Auto Scaling functionality with Apache web servers running on t3.micro instances.

[text](http://autoscalinglabalb-497468542.eu-west-1.elb.amazonaws.com/)

## 🏗️ Infrastructure Overview

### Network Architecture
- **VPC**: Custom Virtual Private Cloud (10.0.0.0/16)
- **Public Subnets**: Two subnets (10.0.1.0/24, 10.0.2.0/24) across different AZs
- **Private Subnets**: Two subnets (10.0.3.0/24, 10.0.4.0/24) across different AZs
- **Internet Gateway**: Provides internet access to public subnets
- **NAT Gateway**: Enables outbound internet access for private subnet instances
- **Route Tables**: Separate routing for public and private subnets

### Load Balancing
- **Application Load Balancer (ALB)**: Distributes traffic across EC2 instances
- **Target Group**: Health checks instances on port 80 with "/" endpoint
- **Listener**: Routes HTTP traffic (port 80) to healthy instances

### Auto Scaling Configuration
- **Launch Template**: Defines instance configuration (t3.micro, Amazon Linux 2)
- **Auto Scaling Group**: 
  - Min: 1 instance
  - Max: 4 instances
  - Desired: 1 instance (initial)
  - Instances deployed in private subnets

## 📊 Auto Scaling Triggers

### Scale-Out Policy (Add Instances)
- **Trigger**: CPU utilization > 30% for 2 consecutive minutes
- **Action**: Add 1 instance
- **Cooldown**: 60 seconds
- **Note**: Meets production-ready threshold requirements

### Scale-In Policy (Remove Instances)
- **Trigger**: CPU utilization < 2% for 10 consecutive minutes
- **Action**: Remove 1 instance
- **Cooldown**: 300 seconds (5 minutes)

## 🖥️ Web Server Setup

Each EC2 instance automatically installs and configures:

### Software Installed
- **Apache HTTP Server**: Web server
- **PHP**: For dynamic content
- **Stress**: CPU stress testing utility

### Web Interface Features
- **Instance Information**: Displays private IP and instance ID
- **Stress Test Button**: Triggers CPU-intensive workload
- **Real-time Status**: Shows stress test progress

### Stress Test Mechanism
When you click the "Stress Test CPU" button:
1. Triggers a CGI script on the server
2. Runs CPU-intensive operations for 60 seconds
3. Uses multiple processes to maximize CPU usage
4. Automatically stops after the timeout period

## 🔐 Security Groups

### ALB Security Group
- **Inbound**: HTTP (port 80) from anywhere (0.0.0.0/0)
- **Outbound**: All traffic allowed

### Instance Security Group
- **Inbound**: 
  - HTTP (port 80) from ALB only
  - SSH (port 22) from specified IP range
- **Outbound**: All traffic allowed

## 🚀 Deployment Instructions

### Prerequisites
- AWS CLI configured with appropriate permissions
- CloudFormation permissions for EC2, VPC, ELB, AutoScaling, CloudWatch

### Deploy the Stack
```bash
aws cloudformation create-stack \
  --stack-name auto-scaling-lab \
  --template-body file://template.yaml \
  --parameters ParameterKey=SSHLocation,ParameterValue=YOUR_IP/32
```

### Access the Application
1. Wait for stack creation to complete (~10-15 minutes)
2. Get the Load Balancer URL from stack outputs
3. Open the URL in your browser
4. You'll see the web interface with instance information

## 🧪 Testing Auto Scaling

### Test Scale-Out (Add Instances)
1. Open the web application in your browser
2. Click the "Stress Test CPU" button
3. Monitor CloudWatch metrics (CPU utilization should spike)
4. Within 2-3 minutes, a new instance should be launched
5. Refresh the page - you may see traffic from the new instance

### Test Scale-In (Remove Instances)
1. Stop clicking the stress test button
2. Wait for CPU utilization to drop below 2%
3. After 10 minutes of low CPU, excess instances will be terminated
4. The ASG will return to minimum capacity (1 instance)

### Monitoring
- **CloudWatch Console**: View CPU metrics and alarm states
- **EC2 Console**: Check Auto Scaling Group activity
- **Load Balancer Console**: Monitor target health

## 📋 Key Components Explained

### Launch Template
Defines the "blueprint" for new instances including:
- AMI (automatically uses latest Amazon Linux 2)
- Instance type (t3.micro for cost efficiency)
- Security groups
- User data script for automatic software installation

### Auto Scaling Group
- Manages the lifecycle of EC2 instances
- Ensures desired capacity is maintained
- Responds to scaling policies triggered by CloudWatch alarms
- Distributes instances across multiple Availability Zones for high availability

### CloudWatch Alarms
- **CPUAlarmHigh**: Monitors average CPU across all instances in the ASG
- **CPUAlarmLow**: Triggers scale-in when CPU usage remains low
- Uses composite metrics to get accurate CPU averages

## 💡 Testing Tips

1. **Multiple Browser Tabs**: Open several tabs and stress test simultaneously for faster scale-out
2. **Patience**: CloudWatch metrics have some delay; scaling actions may take 3-5 minutes
3. **Manual Scaling**: You can manually adjust desired capacity in the ASG console for immediate testing
4. **Cost Awareness**: Remember to delete the stack when finished to avoid charges

## 🧹 Cleanup

To avoid ongoing charges, delete the CloudFormation stack:

```bash
aws cloudformation delete-stack --stack-name auto-scaling-lab
```

This will automatically remove all created resources including:
- EC2 instances
- Load balancer
- NAT Gateway and Elastic IP
- VPC and associated networking components

## 🔍 Troubleshooting

### Common Issues
- **Instances not scaling**: Check CloudWatch alarms and ensure thresholds are met
- **Web page not loading**: Verify security groups and health checks
- **Stress test not working**: Check instance logs in EC2 console

### Useful AWS Console Locations
- **Auto Scaling Groups**: EC2 → Auto Scaling → Auto Scaling Groups
- **CloudWatch Alarms**: CloudWatch → Alarms
- **Load Balancer Health**: EC2 → Load Balancers → Target Groups

## 📝 Notes

- The CPU threshold is intentionally set low (5%) to make testing easier
- In production, typical thresholds would be 70-80% for scale-out
- The template creates a complete isolated environment with proper security
- All instances are placed in private subnets for security best practices
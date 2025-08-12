# AWS Lambda EC2 Start/Stop Scheduler

This project uses **AWS Lambda** and **Amazon EventBridge** to automatically **start** or **stop** EC2 instances on a schedule.  
It helps optimize AWS costs by ensuring instances are only running when required.

---

## 📌 Architecture
- **AWS Lambda** — Executes Python code to start or stop EC2 instances.
- **IAM Role** — Grants Lambda permission to manage EC2 instances.
- **Amazon EventBridge (CloudWatch Events)** — Triggers Lambda functions based on cron schedules (**UTC timezone**).

---

## 🛠 Prerequisites
- AWS account with permission to create IAM roles, Lambda functions, and EventBridge rules.
- Python 3.11 runtime support in Lambda.
- EC2 instance(s) to manage.
- Basic understanding of AWS IAM and Lambda.

---

## 🚀 Setup Instructions

### Step 1 — Create IAM Role for Lambda
1. **Create IAM Policy** `Lambda_StartAndStop`:
   ```json
   {
       "Version": "2012-10-17",
       "Statement": [
           {
               "Sid": "EC2StartStopAccess",
               "Effect": "Allow",
               "Action": [
                   "ec2:StartInstances",
                   "ec2:StopInstances"
               ],
               "Resource": [
                   "arn:aws:ec2:ap-southeast-1:<ACCOUNT_ID>:instance/i-0f8dfb96af197ca2b",
                   "arn:aws:ec2:ap-southeast-1:<ACCOUNT_ID>:instance/i-0e19d1db3f8e3782c"
               ]
           }
       ]
   }
Note: Replace <ACCOUNT_ID> and instance ARNs with your own.

AWS Lambda EC2 Start/Stop Scheduler
===================================

This project uses AWS Lambda and Amazon EventBridge to automatically start or stop EC2 instances on a schedule.
It helps optimize AWS costs by ensuring instances are only running when required.

Architecture:
-------------
- AWS Lambda — Executes Python code to start or stop EC2 instances.
- IAM Role — Grants Lambda permission to manage EC2 instances.
- Amazon EventBridge — Triggers Lambda functions based on cron schedules (UTC timezone).

====================================================================
Step 1: Create IAM Role for Lambda
====================================================================

1. Create IAM Policy "Lambda_StartAndStop":
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "EC2StartStopAccess",
            "Effect": "Allow",
            "Action": [
                "ec2:StartInstances",
                "ec2:StopInstances"
            ],
            "Resource": [
                "arn:aws:ec2:ap-southeast-1:<ACCOUNT_ID>:instance/i-0f8dfb96af197ca2b",
                "arn:aws:ec2:ap-southeast-1:<ACCOUNT_ID>:instance/i-0e19d1db3f8e3782c"
            ]
        }
    ]
}

NOTE: Replace <ACCOUNT_ID> and instance ARNs with your own.

2. Create IAM Role "Ec2_Lambda":
   - Trusted entity: Lambda
   - Attach the policy "Lambda_StartAndStop"

====================================================================
Step 2: Create Lambda Function to Stop EC2 Instances
====================================================================

1. Create Lambda function: ec2-stop-instance
2. Runtime: Python 3.11
3. Attach IAM Role: Ec2_Lambda
4. Code:

import boto3

region = 'ap-southeast-1'
instances = ['i-02d0f769b0f70ff47']  # Replace with your instance IDs
ec2 = boto3.client('ec2', region_name=region)

def lambda_handler(event, context):
    ec2.stop_instances(InstanceIds=instances)
    print(f"Stopped instances: {instances}")

5. Deploy and Test the function.

====================================================================
Step 3: Create Lambda Function to Start EC2 Instances
====================================================================

1. Create Lambda function: ec2-start-instance
2. Runtime: Python 3.11
3. Attach IAM Role: Ec2_Lambda
4. Code:

import boto3

region = 'ap-southeast-1'
instances = ['i-02d0f769b0f70ff47']  # Replace with your instance IDs
ec2 = boto3.client('ec2', region_name=region)

def lambda_handler(event, context):
    ec2.start_instances(InstanceIds=instances)
    print(f"Started instances: {instances}")

5. Deploy and Test the function.

====================================================================
Step 4: Schedule with EventBridge
====================================================================

All AWS cron expressions are in UTC.

Example Schedules:
------------------
- Stop Instance at 9:30 PM IST (16:00 UTC)
  cron(0 16 ? * MON-FRI *)

- Start Instance at 6:30 AM IST (01:00 UTC)
  cron(0 1 ? * MON-FRI *)

Adding a Trigger:
-----------------
1. Open the Lambda function.
2. Click "Add trigger" → EventBridge (Schedule).
3. Select "Create a new rule".
4. Set:
   - Name: ec2-stop-instance
   - Description: Stop EC2 at 9:30 PM IST
   - Schedule: Cron expression above
5. Save and verify the rule in EventBridge.

====================================================================
Best Practices:
====================================================================
- Create separate functions for start and stop operations.
- Use tags on EC2 instances and filter them in Lambda instead of hardcoding IDs.
- Test in non-production before applying to production resources.
- Keep IAM policies least privilege — only allow required EC2 actions and specific resources.

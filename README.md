# Start/Stop EC2 Instances using AWS Lambda and Amazon EventBridge

This guide explains how to automatically start and stop EC2 instances using AWS Lambda functions triggered by Amazon EventBridge rules.

---

## **Prerequisites**
- AWS account with permissions to manage EC2, Lambda, and IAM.
- Instance IDs for the EC2 machines you want to start or stop.
- Time zone for EventBridge rules must be **UTC**.

---

## **Task 1: Create EC2 Instance**
1. Create one Windows or Linux EC2 instance in the AWS Management Console.

---

## **Task 2: Create IAM Role for Lambda**
We will create a role `Ec2_lambda` with permissions to start and stop specific EC2 instances.

### **Step 1 – Create IAM Policy**
Name: **`Lambda_statandstop`**

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "ec2:StartInstances",
                "ec2:StopInstances"
            ],
            "Resource": [
                "arn:aws:ec2:ap-southeast-1:707728974508:instance/i-0f8dfb96af197ca2b",
                "arn:aws:ec2:ap-southeast-1:707728974508:instance/i-0e19d1db3f8e3782c"
            ]
        }
    ]
}
Note: Replace the instance ARN values with your own EC2 instance ARNs.

Step 2 – Create IAM Role
Create a role named Ec2_lambda.

Attach the policy Lambda_statandstop created earlier.

Task 3: Create Lambda Functions
A. Lambda Function – Stop EC2 Instance
Create a new Lambda function named RD_Chatwoot_dev-stop.

Runtime: Python 3.11.

Permissions: Assign the Ec2_lambda role.

Add the following code:

python
Copy
Edit
import boto3
region = 'ap-southeast-1'
instances = ['i-02d0f769b0f70ff47']
ec2 = boto3.client('ec2', region_name=region)

def lambda_handler(event, context):
    ec2.stop_instances(InstanceIds=instances)
    print('Stopped your instances: ' + str(instances))
Note:

Change region and instances values as required.

Multiple instances can be listed as:

python
Copy
Edit
instances = ['i-03ed8bb3a2e2ffb47', 'i-061e8e8245a3f1e15']
Deploy and test the function to verify it stops the instance.

B. Lambda Function – Start EC2 Instance
Create a new Lambda function named RD_Chatwoot_dev-start.

Runtime: Python 3.11.

Permissions: Assign the Ec2_lambda role.

Add the following code:

python
Copy
Edit
import boto3
region = 'ap-southeast-1'
instances = ['i-02d0f769b0f70ff47']
ec2 = boto3.client('ec2', region_name=region)

def lambda_handler(event, context):
    ec2.start_instances(InstanceIds=instances)
    print('Started your instances: ' + str(instances))
Note:
It's best practice to create separate functions for different sets of instances instead of starting multiple at once.

Deploy and test the function to verify it starts the instance.

Task 4: Create EventBridge Rules in Lambda
We will trigger Lambda functions via EventBridge rules directly from Lambda's Add Trigger option.

Rule for Stop Function
Open Lambda function RD_Chatwoot_dev-stop.

Click Add Trigger → EventBridge.

Create a new rule:

Name: RD_Chatwoot_dev-stop

Description: RD_Chatwoot_dev-stop at 9:30 PM IST

Rule type: Schedule expression

Cron expression (UTC):

scss
Copy
Edit
cron(0 16 ? * MON-FRI *)
Save.

Rule for Start Function
Open Lambda function RD_Chatwoot_dev-start.

Click Add Trigger → EventBridge.

Create a new rule:

Name: RD_Chatwoot_dev-start

Description: RD_Chatwoot_dev-start at 6:30 AM IST

Rule type: Schedule expression

Cron expression (UTC):

scss
Copy
Edit
cron(0 1 ? * MON-FRI *)
Example for 9:00 AM IST:

scss
Copy
Edit
cron(30 3 ? * * *)
Save.

Cron Time Conversion
EventBridge cron expressions use UTC time. Convert your IST schedule to UTC before setting rules.

Summary
IAM Role Ec2_lambda allows Lambda to start/stop EC2 instances.

Two Lambda functions are created — one for start, one for stop.

EventBridge rules trigger these Lambda functions on schedule.


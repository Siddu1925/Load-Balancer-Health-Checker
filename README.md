# Load-Balancer-Health-Checker
This scenario helps to check ELB heath with SNS for notifications, The Lambda function to check ELB health, and schedule it to run every 10 minutes with CloudWatch Events. If any instance is unhealthy, you get notified automatically.

# Step-by-Step Process

1. Create an SNS Topic
Go to the SNS dashboard in AWS.
Create a new topic (e.g., ELBHealthAlerts).
Add your email or SMS as a subscription to receive notifications.

2. Create the Lambda Function
Go to the Lambda dashboard.
Create a new function using Python 3.x.
Assign an IAM role with permissions for elasticloadbalancing:DescribeInstanceHealth and sns:Publish.

3. Write the Lambda Code
Use Boto3 to:
Connect to ELB and SNS.
Check the health of instances behind your ELB.
If any are unhealthy, publish a message to your SNS topic.
Paste the provided code into the Lambda editor and update the ELB name and SNS topic ARN.

Code: 
****************************************************************************************************************************
import boto3

def lambda_handler(event, context):
    elb_name = 'your-elb-name'  # Replace with your ELB name
    sns_topic_arn = 'arn:aws:sns:region:account-id:your-topic'  # Replace with your SNS topic ARN

    elb = boto3.client('elb')
    sns = boto3.client('sns')

    # Check instance health
    response = elb.describe_instance_health(LoadBalancerName=elb_name)
    unhealthy = [i for i in response['InstanceStates'] if i['State'] != 'InService']

    if unhealthy:
        message = "Unhealthy instances detected:\n"
        for instance in unhealthy:
            message += f"Instance ID: {instance['InstanceId']}, State: {instance['State']}\n"
        sns.publish(TopicArn=sns_topic_arn, Subject='ELB Health Alert', Message=message)
        print(message)
    else:
        print("All instances are healthy.")
**********************************************************************************************************************************

5. Test the Lambda Function
Manually invoke the Lambda function to ensure it works and sends notifications if unhealthy instances are found.
6. Set Up CloudWatch Event Rule
Go to CloudWatch > Rules.
Create a rule to trigger your Lambda function every 10 minutes (using a schedule expression like rate(10 minutes)).

7. Verify
Confirm you receive SNS notifications when an instance is unhealthy.
Check CloudWatch logs for Lambda execution details.

# 🚀 Serverless EC2 Scheduler

Automate AWS EC2 instance Start/Stop operations using AWS Lambda, EventBridge, IAM, and EC2 Tags.

This project automatically starts development EC2 instances during working hours and stops them after working hours to reduce unnecessary AWS compute costs.

## 🏗️ Architecture

```text
Amazon EventBridge
        │
        │ Scheduled Event
        ▼
    AWS Lambda
        │
        │ Describe / Start / Stop
        ▼
    Amazon EC2
        │
        └── AutoSchedule=true
            Tagged Instances
```

## 🎯 Project Goals

- Automate EC2 Start/Stop operations
- Reduce unnecessary EC2 running costs
- Avoid hardcoding EC2 instance IDs
- Dynamically identify EC2 instances using tags
- Apply least-privilege IAM permissions
- Use EventBridge for scheduled automation
- Build a serverless solution without a dedicated scheduler server

## 🛠️ Technologies Used

- AWS EC2
- AWS Lambda
- Amazon EventBridge
- AWS IAM
- Python
- Boto3
- EC2 Tags

## 🔖 EC2 Tag Configuration

Only EC2 instances with the following tag are managed by the scheduler:

```text
Key   = AutoSchedule
Value = true
```

This allows the Lambda function to dynamically find the required EC2 instances without hardcoding instance IDs.

## ⚙️ How It Works

1. EventBridge reaches the configured schedule.
2. EventBridge invokes the Lambda function.
3. Lambda reads the requested action: `start` or `stop`.
4. Lambda searches for EC2 instances with the `AutoSchedule=true` tag.
5. Lambda collects the matching instance IDs.
6. Lambda starts or stops the instances.
7. Execution information can be reviewed through AWS logging.

## 💻 Lambda Function

```python
import boto3

ec2 = boto3.client('ec2')

def lambda_handler(event, context):

    action = event.get("action")

    response = ec2.describe_instances(
        Filters=[
            {
                'Name': 'tag:AutoSchedule',
                'Values': ['true']
            }
        ]
    )

    instance_ids = []

    for reservation in response['Reservations']:
        for instance in reservation['Instances']:
            instance_ids.append(instance['InstanceId'])

    if not instance_ids:
        return {"message": "No instances found"}

    if action == "start":
        ec2.start_instances(InstanceIds=instance_ids)
        return {"message": f"Started {instance_ids}"}

    elif action == "stop":
        ec2.stop_instances(InstanceIds=instance_ids)
        return {"message": f"Stopped {instance_ids}"}

    else:
        return {"error": "Invalid action"}
```

## 🧪 Lambda Test Events

### Start EC2 Instances

```json
{
  "action": "start"
}
```

### Stop EC2 Instances

```json
{
  "action": "stop"
}
```

## 🔐 IAM Permissions

The Lambda execution role requires:

```text
ec2:DescribeInstances
ec2:StartInstances
ec2:StopInstances
```

Start and stop permissions are restricted to EC2 instances with:

```text
AutoSchedule=true
```

This follows the principle of least privilege.

## 💰 Cost Optimization

Development and testing EC2 instances often do not need to run 24/7.

For example:

```text
Without Scheduler:
24 hours/day

With Scheduler:
12 hours/day

Potential reduction:
~50% EC2 running time
```

Actual savings depend on the EC2 instance type, AWS region, storage, data transfer, and other AWS charges.

## 🔒 Security

The project follows a least-privilege security approach:

- No EC2 instance IDs are hardcoded.
- Start and Stop permissions are restricted using the `AutoSchedule=true` tag.
- Lambda does not require broad `ec2:*` permissions.
- EC2 resources are selected dynamically using tags.
- Lambda execution logs can be monitored using CloudWatch.

> **Important:** Protect permissions that allow users to modify the `AutoSchedule` tag because changing the tag can change which EC2 instances are controlled by the scheduler.

## 📊 Advantages

- ⚡ **Serverless** — No scheduler server required
- 🤖 **Automated** — EC2 operations happen automatically
- 🔄 **Dynamic** — Uses tags instead of hardcoded instance IDs
- 🔐 **Secure** — IAM permissions can be restricted
- 💰 **Cost-effective** — Reduces unnecessary EC2 running time
- 📈 **Scalable** — Can manage multiple tagged EC2 instances

## 📁 Project Structure

```text
serverless-ec2-scheduler/
│
├── lambda_function.py
├── iam-policy.json
└── README.md
```

## 🔮 Future Improvements

- CloudWatch alarms
- SNS notifications
- Better monitoring
- Weekend scheduling
- Time-zone-aware scheduling
- Additional EC2 tagging rules
- Advanced cost-control policies

## 🎓 What I Learned

Through this project, I practiced:

- AWS Lambda
- Amazon EC2
- Amazon EventBridge
- AWS IAM
- IAM least-privilege policies
- EC2 resource tagging
- Python Boto3
- Serverless architecture
- Event-driven automation
- AWS cost optimization

## 👨‍💻 Author

**Ketan Kolambe**

Cloud / DevOps Enthusiast

AWS | Terraform | Linux | Git | GitHub | Python

---

⭐ If you found this project useful, consider giving it a star!

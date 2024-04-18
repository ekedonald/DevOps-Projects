# Monitoring Infrastructure in AWS (CloudWatch)
Monitoring refers to the process of observing and collecting data about the performance, health and behaviour of systems, applications, networks or infrastructure components. The primary goal of monitoring is to ensure that these systems operate effectively, efficiently and securely while also detecting and addressing any issues or anomalies in a timely manner.

## AWS CloudWatch and CloudTrail
AWS CloudWatch is a monitoring and observabality service provided by Amazon Web Services (AWS). It allows users to collect and track metrics, monitor logs, set alarms and automatically react to changes in AWS resources and applications running on the AWS infrastructure. 

CloudWatch provides insights into the performance, health and operational status of AWS resources and applications helping users to troubleshoot issues, optimise resource utilisation and ensure the reliability of their systems.

AWS CloudTrail enables governance, compliance, operational auditing and risk auditing of your AWS account.CloudTrail records and logs all API activity in your AWS account providing a comprehensive trail of events that can be used for security analysis, resource change tracking, troubleshooting and compliance auditing.

## CloudWatch Metrics and Alarms
Amazon CloudWatch Metrics and Alarms are essential components of the Amazon CloudWatch service which provides monitoring and observability capabilities for AWS resources and applications. Let's delve into each of these concepts:

### CloudWatch Metrics
CloudWatch Metrics are data points representing the behaviour of AWS resources and applications over time. These metrics can be collected from various AWS services such as Amazon EC2, Amazon RDS, Amazon S3, AWS Lambda, and many others. 

Metrics provide insights into the performance, health, and operational status of these resources, allowing users to monitor and analyse their behaviour.

![image-1](./images/0.%20imag-2.png)

Key aspects of CloudWatch Metrics include:

**Default and Custom Metrics**: AWS services automatically publish default metrics to CloudWatch, such as CPU utilisation, network traffic, and disk 1/O for EC2 instances. Additionally, users can create custom metrics to monitor specific aspects of their applications or services.

**Namespace and Dimensions**: Metrics are organised into namespaces, which categorise related metrics together. Within each namespace, metrics can have dimensions that further specify the resource or aspect being monitored. For example, an EC2 instance metric might have dimensions such as **Instanceld** or **InstanceType**.

**Timestamps and Units**: Each metric data point includes a timestamp indicating when the measurement was taken, as well as a unit specifying the measurement's scale (e.g. bytes, percentage, seconds).

![image3](./images/0.%20imag-3%20.png)

**Retention and Granularity**: CloudWatch retains data for different periods depending on the data's age and granularity. Users can specify the granularity of their metric data, ranging from one-minute to one-day intervals.

![image1](./images/0.%20imag-1.png)

### CloudWatch Alarms
CloudWatch Alarms allow users to define thresholds on CloudWatch Metrics and trigger actions when these thresholds are breached. Alarms are used to proactively monitor the health and performance of AWS resources and applications, enabling users to respond promptly to changes in their environment.

Key aspects of CloudWatch Alarms include:

**Thresholds and Actions**: Users can set thresholds on CloudWatch Metrics, specifying conditions that, when met or exceeded, trigger alarm states. When an alarm enters an alarm state, users can configure actions such as sending notifications via Amazon SNS, executing AWS Lambda functions, or auto-scaling resources.

**Alarm States**: CloudWatch Alarms have three possible states: OK, INSUFFICIENT_DATA, and ALARM. The OK state indicates that the metric is within the defined threshold, while the ALARM state indicates that the threshold has been breached. The INSUFFICIENT DATA state occurs when there is not enough data to evaluate the alarm.

**Alarm History**: CloudWatch maintains a history of alarm state changes, allowing users to track when alarms transition between states and investigate the circumstances surrounding each state change.

**Configuration and Management**: Users can create, modify, and delete alarms through the CloudWatch Management Console, AWS CLI, or SDKs. Alarms can be managed individually or as part of larger monitoring configurations, such as CloudFormation templates or AWS Auto Scaling policies.


## Monitoring AWS EC2 using CloudWatch
The following steps are taken to monitor an AWS EC2 Instance using CloudWatch:

### Step 1: Create an IAM Role With CloudWatchFull Access and SSM FullAccess
1. Navigate to your console and click on IAM.

![click IAM](./images/1.%20click%20on%20IAM.png)

2. Click on Roles.

![roles](./images/1.%20click%20on%20Roles.png)

3. Click on the `Create role` button.

![create role](./images/1.%20click%20on%20create%20role.png)

4. Choose the following parameters and click on `Next`:

* Trusted entity type: AWS service
* Service or use case: EC2
* Use case: EC2

![parameters](./images/1.%20create%20role%20parameters.png)

5. Search and select the following permission policies: `CloudWatchFullAccess` and `AmazonSSMFullAccess` for your role and click on `Next`.

![cloudwatchfullaccess](./images/1.%20cloudwatchfullaccess%20policy.png)
![amazonssmfullaccess](./images/1.%20ssmfullaccess%20policy.png)

6. Name your role `EC2-Role-2` and click on `Create role`.

![name role](./images/1.%20role%20name%20.png)

### Step 2: Create a Parameter in System Manager
1. Navigate to your console, search for `Systems Manager` and click on it.

![systems manager](./images/2.%20systems%20manager.png)

2. Click on the `Parameters Store` tab.

![parameters store](./images/2.%20parameters%20store%20tab.png)

3. Click on `Create parameter`.

![create parameter](./images/2.%20create%20parameter.png)

4. Name it `EC2_Tag` and paste the code below into the `Value` tab:

```sh
{
	"metrics": {
		"append_dimensions": {
			"InstanceId": "${aws:InstanceId}"
		},
		"metrics_collected": {
			"mem": {
				"measurement": [
					"mem_used_percent"
				],
				"metrics_collection_interval": 180
			},
            "disk": {
				"measurement": [
                     "disk_used_percent"
				],
				"metrics_collection_interval": 180
			}
		}
	}
}
```

![name parameter and paste value](./images/2.%20name%20parameter%20and%20paste%20value.png)

The parameters above are a configuration file for the CloudWatch agent which defines the metrics that will be collected from your EC2 instance and sent to CloudWatch.

* `"metrics"`: This is the top-level key in the configuration file indicating that it contains the definitions for the metrics to be collected.

* `"append_dimensions"`: This section specifies dimensions to be appended to all collected metrics. Dimensions are key-value pairs that help identify the source of the data in CloudWatch. In this case, the dimension **"InstanceId"** is appended and its value is populated with the instance ID of the EC2 instance where the CloudWatch agent is installed.

* `"InstanceId:" "${aws:InstanceId}"`: This line specifies that the value of the **"InstanceId"** dimension should be dynamically populated with the Instance ID of the EC2 instance.

* `"metrics_collected"`: This section defines the specific metrics to be collected from the EC2 instance.

* `"mem"`: This subsection specifies memory-related metrics to be collected.

* `"measurement" `: This is an array of specific memory metrics to collect. In this case, only.

* `"mem_used_percent"`: is specified, which represents the percentage of memory used on the instance.

* `"metrics_collection_interval"`: This parameter specifies how frequently (in seconds) the metrics
should be collected. Here, memory metrics will be collected every 60 seconds.

* `"disk"`: This subsection specifies disk-related metrics to be collected.

* `"measurement"`: This is an array of specific disk metrics to collect. Only `"disk_used_percent"` is specified, representing the percentage of disk space used on the instance.

* `"metrics_collection_interval"`: Similar to the memory section, this parameter specifies how frequently disk metrics will be collected, which is every 60 seconds.

### Step 3: Create an EC2 Instance and attach the EC2-Role created
1. Click on `Launch instance`.

![launch instance](./images/3.%20launch%20instance.png)

2. Name and select the following parameters:

* Name: CloudWatch
* AMI: Amazon Linux 2023 AMI
* keypair: *choose your own*
* Security Group: *any security group that allows SSH connection from any IPv4 address*

![instance parameters 1](./images/3.%20instance%20parameters1.png)
![instance parameters 2](./images/3.%20instance%20parameters2.png)

3. Scroll down, click on the `Advanced details` tab and select the `EC2_Role` as the **IAM instance profile**.

![advanced details](./images/3.%20advnaced%20details.png)

4. Click on the `Instance ID` of the EC2 Instance.

![instance id](./images/3.%20instance%20id.png)

5. Click on `Connect` to connect to the instance.

![connect1](./images/3.%20connect.png)
![connect2](./images/3.%20connect2.png)

6. Create a file `script.sh` to install the **CloudWatch** agent.

```sh
vi script.sh
```

7. Paste the shell script below into the file:

```sh
#!/bin/bash
wget https://s3.amazonaws.com/amazoncloudwatch-agent/linux/amd64/latest/AmazonCloudWatchAgent.zip
unzip AmazonCloudWatchAgent.zip
sudo ./install.sh
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a fetch-config -m ec2 -c ssm:/alarm/AWS-CWAgentLinConfig -s
```

![paste the script](./images/3.%20paste%20the%20script.png)

8. Make the file executable using the command shown below:

```sh
sudo chmod +x script.sh
```

![executable](./images/3.%20executable.png)

9. Run the script.

```sh
./script.sh
```

![run the script](./images/3.%20run%20the%20script.png)

10. Start the CloudWatch agent

```sh
 sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -m ec2 -a start
```

![start cloudwatch agent](./images/3.%20start%20cloudwatch%20agent.png)

#### Step 4: Monitor your Metric in CloudWatch
Before the EC2 instance can be monitored, a new policy must be created and attached to the IAM role so the role has the necessary permissions to scrape data.

1. On your console home, click on IAM.

![IAM](./images/1.%20click%20on%20IAM.png)

2. CLick on `Policies` and `Create policy`.

![policies and create policy](./images/4.%20policies%20and%20create%20policy.png)

3. Paste the code below into the **JSON Policy editor**.

```sh
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ec2:DescribeTags"
            ],
            "Resource": "*"
        }
    ]
}
```

![paste the code](./images/4.%20paste%20the%20code.png)

4. Name the policy **EC2_Policy_Tag** and click on `Create policy`.

![name the policy](./images/4.%20name%20policy%20and%20create%20policy.png)

5. Click on `Roles` and `EC2_Role`.

![roles and EC2_Role](./images/4.%20click%20roles%20and%20ec2_roles.png)

6. Click on `Add permissions` and `Attach policies`.

![add permissions and attach policies](./images/4.%20add%20permissions%20and%20attach%20policies.png)

7. Search for `EC2_Policy`, select it and click `Add permssions`.

![search ec2_policy and add permissions](./images/4.%20search%20ec2_policy%20and%20add%20permissions.png)

8. On your console home, search for `CloudWatch` and click on it.

![search cloudwatch](./images/4.%20search%20cloudwatch.png)

9. Click on `All metrics` tab.

![all metrics](./images/4.%20click%20on%20all%20metrics.png)

10. Click on `CWAgent`.

![cwagent](./images/4.%20CWAgent.png)

11. Select either of the 2 tabs shown below:

![select tabs](./images/4.%20select%20either%20tab.png)

12. Click on the `CloudWatch` EC2 Instance.

![cloudwatch](./images/4.%20cloudwatch%20instance.png)
**Send Nginx Logs to AWS Cloudwatch**

**- Step-1 : Attaching an IAM Role to instance**

Open the IAM console at `https://console.aws.amazon.com/iam/`

In the navigation pane, choose `Roles`

Choose the role by selecting the role name (do not select the check box next to the name).

Choose `Attach Policies`, `Create Policy.`

Choose the JSON tab and type the following JSON policy document.

```
					{
		  "Version": "2012-10-17",
		  "Statement": [
		    {
		      "Effect": "Allow",
		      "Action": [
		        "logs:CreateLogGroup",
		        "logs:CreateLogStream",
		        "logs:PutLogEvents",
		        "logs:DescribeLogStreams"
		    ],
		      "Resource": [
		        "*"
		    ]
		  }
		 ]
		}
```
On the Review Policy page, type a Name and a Description (optional) for the policy that you are creating. Review the policy Summary to see the permissions that are granted by your policy. Then choose Create policy to save your work.


Close the browser tab or window, and return to the `Add permissions` page for your role. Choose `Refresh`, and then choose the new policy to attach it to your role.

Choose `Attach Policy.`


**- Step 2: Install CloudWatch Logs Agent**

Download the file into user home folder:
```
cd ~ 

curl https://s3.amazonaws.com/aws-cloudwatch/downloads/latest/awslogs-agent-setup.py -O
```
You need to install `python` if you haven’t before running this command:

```
	sudo python ./awslogs-agent-setup.py --region us-east-1
```

Once installed you will be asked by the following options which almost of options you can just skip by pressing enter except the AWS credential keys that you need to input correctly :	


**- Step 3: Configure CloudWatch Logs Agent**

 Edit `/var/awslogs/etc/awslogs.conf` file which will have default log path from previous step, you can just remove that and only have `NGINX` logs path setup as shown below:

 	nginx access.log path: `/var/log/nginx/access.log`

 	nginx error.log path:  `/var/log/nginx/error.log`

 Change the `awslogs.conf` to:

``` 
		[/var/log/nginx/access.log]
		datetime_format = %d/%b/%Y:%H:%M:%S %z
		file = /var/log/nginx/access.log
		buffer_duration = 5000
		log_stream_name = access.log
		initial_position = end_of_file
		log_group_name = /ec2/nginx/logs

		[/var/log/nginx/error.log]
		datetime_format = %Y/%m/%d %H:%M:%S
		file = /var/log/nginx/error.log
		buffer_duration = 5000
		log_stream_name = error.log
		initial_position = end_of_file
		log_group_name = /ec2/nginx/logs
```

 Last step is by restarting the service:

 	`sudo service awslogs restart`

You can monitor the log from `/var/log/awslogs.log` file.


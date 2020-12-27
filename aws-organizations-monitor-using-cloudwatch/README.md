# AWS Organizations and Monitor important changes to your organization with CloudWatch Events

# A. AWS Organizations
# B. Monitor important changes to your organization with CloudWatch Events

# A. AWS Organizations

## Prerequisites
- Access to two existing AWS accounts (you create a third as part of this tutorial) and that you can sign in to each as an administrator.
- accounts
	111111111111 email : management_email@rediffmail.com (management account) 							    770239628111 
	222222222222 email : email2@yahoo.com     	(invite this account to join organization )			793413835222 
	333333333333 email : email3@gmail.com   	(create as a member of the organization)			793413835333
	 
## Steps:
## Step 1: Create your organization

### 1.1. Invite an existing account to join your organization
	
	Organization ID			: o-5xnyaemxvr
	Management account email: management_email@rediffmail.com
	
### 1.2. Create a member account

	 Account name : email2333     
	 Email address associated with the account : email3@gmail.com
	 IAM role name : AWS Organizations creates "OrganizationAccountAccessRole" role to grant the organization full administrative control over the new account.
   	
## Step 2: Create the organizational units

	AWS Organizations console -> Organize Accounts tab ->  New organizational unit
    Production 
	MainApp
	
	Root- > 111111111111
	Production OU ->  222222222222 account
		MainApp OU -> 333333333333 account
		
		
## Step 3: Create the service control policies
A service control policy (SCP) is a type of policy that you can use to centrally control the maximum available permissions for all accounts in your organization. 
Using SCPs, you can ensure that your accounts stay within your organization’s access control guidelines

create three service control policies (SCPs) and attach them to the root and to the OUs to restrict what users in the organization's accounts can do
The first SCP prevents anyone in any of the member accounts from creating or modifying any AWS CloudTrail logs that you configure. 
The management account isn't affected by any SCP, so after you apply the CloudTrail SCP, you must create any logs from the management account

### 3.1. Create the first SCP that blocks CloudTrail configuration actions
	
AWS Organizations-> Policies -> Create policy
Policy name = Block CloudTrail Configuration Actions
Policy section on the left, select CloudTrail for the service
Then choose the following actions: AddTags, CreateTrail, DeleteTrail, RemoveTags, StartLogging, StopLogging, and UpdateTrail
choose Add resource and specify CloudTrail and All Resources. Then choose Add resource.
Choose Create policy

The policy statement looks like below
```
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Sid": "Statement1",
			"Effect": "Deny",
			"Action": [
				"cloudtrail:AddTags",
				"cloudtrail:CreateTrail",
				"cloudtrail:DeleteTrail",
				"cloudtrail:RemoveTags",
				"cloudtrail:StartLogging",
				"cloudtrail:StopLogging",
				"cloudtrail:UpdateTrail"
			],
			"Resource": [
				"*"
			]
		}
	]
}
```

### 3.2. create the second policy that allows approved services for the production OU
The second policy defines an allow list of all the services and actions that you want to enable for users and roles in the Production OU. 
When you're done, users in the Production OU can access only the listed services and actions.

AWS Organizations-> Policies -> Create policy
Policy name = Allow List for All Approved Services
Policy  Section, add below and Choose Create policy

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "Stmt1111111111111",
            "Effect": "Allow",
            "Action": [ 
                "ec2:*",
                "elasticloadbalancing:*",
                "codecommit:*",
                "cloudtrail:*",
                "codedeploy:*"
              ],
            "Resource": [ "*" ]
        }
    ]
}
```

### 3.3. create the third policy that denies access to services that can't be used in the MainApp OU
The final policy provides a deny list of services that are blocked from use in the MainApp OU. 
For this tutorial, you block access to Amazon DynamoDB in any accounts that are in the MainApp OU

AWS Organizations-> Policies -> Create policy
Policy name = Deny List for MainApp Prohibited Services
Policy section on the left, select Amazon DynamoDB for the service. For the action, choose All actions.

Still in the left pane, choose Add resource and specify DynamoDB and All Resources. Then choose Add resource.

Create policy to save the SCP

The policy statement looks similar to the following.
```
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Sid": "Statement1",
			"Effect": "Deny",
			"Action": [
				"dynamodb:*"
			],
			"Resource": [
				"*"
			]
		}
	]
}
```

### 3.4. Enable the service control policy type in the root
To enable SCPs for your root
Organize accounts tab-> choose your root.
In the Details pane on the right, under ENABLE/DISABLE POLICY TYPES and next to Service control policies, choose Enable.

### 3.5. Attach the policies to the root and the OUs
- Organize accounts tab-> in the Details pane on the right-> under POLICIES-> choose SERVICE CONTROL POLICIES.
- Choose Attach next to the SCP named Block CloudTrail Configuration Actions to prevent anyone from altering the way that you configured CloudTrail. In this tutorial, you attach it to the root so that it affects all member accounts.
- Choose the Production OU (not the check box) to navigate to its contents.
- Under POLICIES -> choose SERVICE CONTROL POLICIES -> then choose Attach next to Allow List for All Approved Services to enable users or roles in member accounts in the Production OU to access the approved services.
- The information pane now shows that two SCPs are attached to the OU: the one that you just attached and the default FullAWSAccess SCP. However, because the FullAWSAccess SCP is also an allow list that allows all services and actions, you must detach this SCP to ensure that only your approved services are allowed.
The Details pane now shows by highlighting that two SCPs are attached to the root: the one you just created and the default FullAWSAccess SCP.
- To remove the default policy from the Production OU, next to FullAWSAccess, choose Detach. After you remove this default policy, all member accounts under the root immediately lose access to all actions and services that are not on the allow list SCP that you attached in the preceding step. Any requests to use actions that aren't included in the Allow List for All Approved Services SCP are denied. This is true even if an administrator in an account grants access to another service by attaching an IAM permissions policy to a user in one of the member accounts.
- Now you can attach the SCP named Deny List for MainApp Prohibited services to prevent anyone in the accounts in the MainApp OU from using any of the restricted services.
To do this, choose the MainApp OU (not the check box) to navigate to its contents.
- In the Details pane, under POLICIES, expand the Service control policies section. In the list of available policies, next to Deny List for MainApp Prohibited Services, choose Attach.

## Step 4: Testing your organization's policies

- If you sign in as a user in the management account, you can perform any operation that is allowed by your IAM permissions policies. The SCPs don't affect any user or role in the management account, no matter which root or OU the account is located in
- If you sign in as the root user or an IAM user in account 222222222222, you can perform any actions that are allowed by the allow list. AWS Organizations denies any attempt to perform an action in any service that isn't in the allow list. Also, AWS Organizations denies any attempt to perform one of the CloudTrail configuration actions.
- If you sign in as a user in account 333333333333, you can perform any actions that are allowed by the allow list and not blocked by the deny list. AWS Organizations denies any attempt to perform an action that isn't in the allow list policy and any action that is in the deny list policy. Also, AWS Organizations denies any attempt to perform one of the CloudTrail configuration actions.


# B. Monitor important changes to your organization with CloudWatch Events

Configure CloudWatch Events to monitor your organization for changes. 
Configuring a rule that is triggered when users invoke specific AWS Organizations operations. 
Configure CloudWatch Events to run an AWS Lambda function when the rule is triggered, and 
Configure Amazon SNS to send an email with details about the event.

## Prerequisites
- management account (formerly known as the "master account") in your organization. 
- Or The IAM user must have permissions to create and configure a log in CloudTrail, a function in Lambda, a topic in Amazon SNS, and a rule in CloudWatch.
- Access to an existing Amazon Simple Storage Service (Amazon S3) bucket (or you have permissions to create a bucket) to receive the CloudTrail log that you configure in step 1

```
note: 
Currently, AWS Organizations is hosted in only the US East (N. Virginia) Region (even though it is available globally). To perform the steps in this tutorial, you must configure the AWS Management Console to use that region.
```

## Steps: 
## Step 1: Configure a trail and event selector
Create a log, called a trail, in AWS CloudTrail. You configure it to capture all API calls.

Configure a log (called a trail) in AWS CloudTrail. 
Also configure an event selector on the trail to capture all read/write API calls so that CloudWatch Events has calls to trigger on.

- Open CloudTrail  and select  the US East (N. Virginia) Region
- choose Trails and Create trail
- Trail name = My-Test-Trail (Enter a new S3 bucket name and folder (prefix) to store your logs. Bucket names must be globally unique)
```
note : 
Trail log bucket and folder = aws-cloudtrail-logs-770239628917-c0b0ada8
Logs will be stored in aws-cloudtrail-logs-770239628917-c0b0ada8/AWSLogs/770239628917
```
- Choose the trail My-Test-Trail that you just created.
- Choose the Edit icon next to Management events.
- For Read/Write events, choose All, choose Save, and then choose Configure


```
CloudWatch Events enables you to choose from several different ways to send alerts when an alarm rule matches an incoming API call. 
This tutorial demonstrates two methods: 
invoking a Lambda function that can log the API call and 
sending information to an Amazon SNS topic that sends an email or text message to the topic's subscribers. 

In the next two steps, you create the components you need: 
the Lambda function, and 
the Amazon SNS topic.
```

## Step 2: Configure a Lambda function
Create an AWS Lambda function that logs details about the event to an S3 bucket.
Create a Lambda function that logs the API activity that is sent to it by the CloudWatch Events rule that you configure later
- Open the AWS Lambda console
- New to Lambda, choose Get Started Now on the welcome page; otherwise, choose Create a function.
- Create function page, choose Blueprints
- Blueprints search box, enter hello for the filter and choose the hello-world blueprint
- Choose Configure
- On the Basic information page
	- Lambda function name=LogOrganizationEvents
	- For Role, choose Create a custom role and then, at the bottom of the AWS Lambda requires access to your resources page, choose Allow. 
	This role grants your Lambda function permissions to access the data it requires and to write its output log.
	- Choose Create function.
	
	
-  edit the code for the Lambda function
```
console.log('Loading function');

exports.handler = async (event, context) => {
    console.log('LogOrganizationsEvents');
    console.log('Received event:', JSON.stringify(event, null, 2));
    return event.key1;  // Echo back the first key value
    // throw new Error('Something went wrong');
};
```

## Step 3: Create an Amazon SNS topic that sends emails to subscribers
Create an Amazon SNS topic that sends emails to its subscribers, and then subscribe yourself to the topic.

In this step, create an Amazon SNS topic that emails information to its subscribers. 
You make this topic a "target" of the CloudWatch Events rule that you create later.

- Open the Amazon SNS console and Choose Create new topic
- Topic name = OrganizationsCloudWatchTopic
```
note: To use this topic with SMS subscriptions, enter a display name. Only the first 10 characters are displayed in an SMS message
```
- Display name - optional= OrgsCWEvnt
- Choose Create topic.
- Now you can create a subscription for the topic. Choose the ARN for the topic that you just created
- Choose Create subscription
- Protocol = Email
- Endpoint = enter your email address.
- Choose Create subscription. 

```
note: 
AWS sends an email to the email address that you specified in the preceding step. Wait for that email to arrive, and then choose the Confirm subscription link in the email to verify that you successfully received the email

Email: 
You have chosen to subscribe to the topic:
arn:aws:sns:us-east-1:770239628917:OrganizationsCloudWatchTopic

To confirm this subscription, click or visit the link below (If this was in error no action is necessary):
Confirm subscription

Subscription confirmed!
You have subscribed email2@gmail.com to the topic:
OrganizationsCloudWatchTopic.

Your subscription's id is:
arn:aws:sns:us-east-1:770239628917:OrganizationsCloudWatchTopic:fe542cb3-4610-42e3-baf9-36457d2517e0
```

- Return to the console and refresh the page. The Pending confirmation message disappears and is replaced by the now valid subscription I


## Step 4: Create a CloudWatch Events rule
Create a rule that tells CloudWatch Events to pass details of specified API calls to the Lambda function and to SNS topic subscribers.
Now that the required Lambda function exists in your account, you create a CloudWatch Events rule that invokes it when the criteria in the rule are met.

- Open  CloudWatch console and choose the US East (N. Virginia) Region
- choose Rules and then choose Create rule
- For Event source, do the following
	- Choose Event pattern.
	- Choose Build event pattern to match events by service.
	- Service Name =  Organizations.
	- Event Type = AWS API Call via CloudTrail.
	- Choose Specific operation(s) and then enter the APIs that you want monitored: CreateAccount and CreateOrganizationalUnit. 
	```
	note: 
	You can select any others that you also want. For a complete list of available AWS Organizations APIs, 
	see the [AWS Organizations API Reference](https://docs.aws.amazon.com/organizations/latest/APIReference/)
	```
- Under Targets, for Function, choose the function that you created in the previous procedure e.g. LogOrganizationEvents
- Under Targets, choose Add target.
- In the new target row, choose the dropdown header and then choose SNS topic
- For Topic, choose the topic named = OrganizationCloudWatchTopic that you created in the preceding procedure.
- Choose Configure details.
- On the Configure rule details page, Name = OrgsMonitorRule, leave State selected and then choose Create rule.

## Step 5: Test your CloudWatch Events rule
Test your new rule by running one of the monitored operations. In this tutorial, the monitored operation is creating an organizational unit (OU). You view the log entry that the Lambda function creates, and you view the email that Amazon SNS sends to subscribers.

### In this step, you create an organizational unit (OU) and observe the CloudWatch Events rule generate a log entry and send an email to you with details about the event.
- Open the AWS Organizations console -> Organize accounts tab ->  choose New organizational unit.
- name = TestCWEOU
- choose Create organizational unit

### To see the CloudWatch Events log entry

- open CloudWatch console -> choose Logs	
- Under Log Groups, choose the group that is associated with your Lambda function: /aws/lambda/LogOrganizationEvents
- Each group contains one or more streams, and there should be one group for today. Choose it
```
2020/12/14/[$LATEST]2fa11610a5de49708df631c8312af575
```
- View the log. You should see rows similar to the following.
```
Timestamp 						Message 
								No older events at this moment. Retry
2020-12-14T19:17:37.463+01:00	START RequestId: 7fedb7f7-bfea-41dc-b012-30507e48ffae Version: $LATEST
2020-12-14T19:17:37.463+01:00	2020-12-14T18:17:37.462Z undefined INFO Loading function
2020-12-14T19:17:37.486+01:00	2020-12-14T18:17:37.469Z 7fedb7f7-bfea-41dc-b012-30507e48ffae INFO value1 = undefined
2020-12-14T19:17:37.486+01:00	2020-12-14T18:17:37.486Z 7fedb7f7-bfea-41dc-b012-30507e48ffae INFO value2 = undefined
2020-12-14T19:17:37.486+01:00	2020-12-14T18:17:37.486Z 7fedb7f7-bfea-41dc-b012-30507e48ffae INFO value3 = undefined
2020-12-14T19:17:37.487+01:00	END RequestId: 7fedb7f7-bfea-41dc-b012-30507e48ffae
2020-12-14T19:17:37.487+01:00	REPORT RequestId: 7fedb7f7-bfea-41dc-b012-30507e48ffae Duration: 19.72 ms Billed Duration: 20 ms Memory Size
```

- Select the middle row of the entry to see the full JSON text of the received event. 
You can see all the details of the API request in the requestParameters and responseElements pieces of the output.
- Check your email account for a message from OrgsCWEvnt (the display name of your Amazon SNS topic). 
The body of the email contains the same JSON text output as the log entry that is shown in the preceding step.

## Clean up
```
CloudTrail console (step 1) -> delete the trail named =  My-Test-Trail    
S3 bucket (step 1)- > 
Lambda console ( step 2) ->  delete the function named = LogOrganizationEvents 
 Amazon SNS console (step 3)- >  delete the Amazon SNS topic named = OrganizationsCloudWatchTopic
 CloudWatch console (step 4) - > delete the CloudWatch rule named = OrgsMonitorRule
```


Summary :
You configured CloudWatch Events to monitor your organization for changes. 
You configured a rule that is triggered when users invoke specific AWS Organizations operations. 
The rule ran a Lambda function that logged the event and sent an email that contains details about the event.


## References
### https://docs.aws.amazon.com/organizations/latest/userguide/orgs_tutorials_basic.html
### https://docs.aws.amazon.com/organizations/latest/userguide/orgs_tutorials_cwe.html
### other more: https://docs.aws.amazon.com/organizations/latest/userguide/orgs_security_incident-response.html
### Best practices for AWS Organizations: https://docs.aws.amazon.com/organizations/latest/userguide/orgs_best-practices.html

##  Next
### Cloud trail : https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-tutorial.html
### CloudWatch Tutorials : https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch-tutorials.html
### landa fiunction tutorial : https://docs.aws.amazon.com/lambda/latest/dg/getting-started.html





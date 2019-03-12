# Automated Monitoring for Security Misconfigurations with Security Monkey

![Netlfix Ape Guy](assets/security_monkey.jpg)

If you are working on cloud technologies and specifically cloud security, the first few questions you would get should be around security, data safety, cost effectiveness etc. Additionally, companies in the FinTech and Healthcare sectors may be concerned about how secure cloud technologies can be for them.

Based on my considerable experience and battle scars, I would recommend looking at Security Monkey which was open sourced in 6/30/2014.


### So, What is Security Monkey?

Security Monkey is an OpenSource application from Netflix which monitors, alerts and reports one or multiple AWS accounts for anomalies. Security Monkey can run on an Amazon EC2 (AWS) instance, Google Cloud Platform (GCP) instance (Google Cloud Platform), or OpenStack (public or private cloud) instance. While Security Monkey's main purpose is security, it also proves a useful tool for tracking down potential problems as it is essentially a change tracking system.

Security Monkey has been an invaluable tool that you will honestly end up using everyday. Referencing here is my chrome browser history reflecting my usage statistics,

> TIP : Script to import Chrome Browser history to Elastic Search https://github.com/nagwww/chrome-history

![Chrome Usage](assets/chrome_usage.png)

Here are some common scenarios where Security Monkey can be of help, especially in a multi-account environment:

## The Multi-Account Services; A bulleted list


### Security Groups

- Generates an Audit report of all the issues (IE, Security groups which are wide open to the internet or ingress from 0.0.0.0/0, etc.)
- Creates an email alert when security group changes are done, which can come in handy when you have a PCI/SOX/HIPPA compliance related environment.
- Alerts you when a user/developer adds 0.0.0.0/0 to a security group.
- Searches for particular IP/CIDR blocks which is really helpful if you have multiple AWS accounts.
- Helps you identify the Security group name given the security group ID. This is helpful since for cross-account security group access, AWS now no longer shows the security group name, but does show the ID.
- Historical Information : Security Monkey acts as the source control for your security groups. For instance, to know the state of a security group from a month ago, one can go back and perform a diff of the current state.
- Keeps track of your PCI/SOX/HIPPA compliant environment for changes, where you can set an alert to email when a change is performed to a security group or send it to your internal auditor/change control management team directly.
- Alerts when a new Security Group is created.
- Helps locate a security group which no longer exists in AWS or was deleted knowing or unknowingly.


### Simple Storage Service

- Security Monkey acts as the source control for your S3 buckets policies, ACL, lifecycle rules.
- Generates an audit report of all the current issues (IE, AWS S3 buckets which are accessible to everyone shared across unknown AWS accounts and have conditional statements)
- Creates an e-mail alert when a S3 bucket is added or deleted.
- AWS S3 resource policies are used to grant fine grain access controls for S3 buckets and objects. All the ACLs and policies are stored in security monkey which triggers alerts when changes are done. Comes handy when you have sensitive S3 buckets and you want to monitor for changes.
- Tracks S3 buckets for bucket-level encryption.
- Tracks versioning of buckets.
- Tracks the lifecycle object of an S3 bucket. Lifecycle rules enable you to automatically archive/delete S3 objects based on predefined rule sets.
- Monitors S3 ACLs and bucket policies since last check and alerts when buckets are publicly accessible.

Here is a good read on the [100s AWS S3 buckets left open exposing private data](https://www.helpnetsecurity.com/2013/03/27/thousands-of-amazon-s3-buckets-left-open-exposing-private-data/).


### Identity and Access Management

- Generates a report of all active IAM users with active access keys.
- Lists all active IAM active keys which are not rotated in the last 90 days.
- Lists all inactive access keys which can be used to clean up.
- Lists all the active keys which were not used in the last 90 days.
- Lists IAM User who have AWS Console access, however with no MFA enabled.
- Alerts when an IAM Role has full Admin privileges.
- Finally Security Monkey also acts as a source control for all your IAM policies attached to the users/roles.

A good read on the [AWS Console breach](http://arstechnica.com/security/2014/06/aws-console-breach-leads-to-demise-of-service-with-proven-backup-plan).


### Elastic Loadbalancers

- Alerts when an ELB is internet facing
- Alerts when ELB logging is not enabled.
- Alerts when deprecated ciphers are enabled on an ELB.
- Provides a list of the weak ciphers if enabled on the ELB policy.


### Simple Email Service

- Monitors SES identities to make sure only valid company email address are configured as verified.
- Monitors for all SES objects that are not verified and can be cleaned up.


### Simple Queue Service

- Alerts when an SQS queue has a policy granting access to everyone or open to world.
- Notifies when there is change to the SQS policy.
- Historical Information: Security Monkey is like the source control for SQS resource policies. To know the state of an SQS policy from a month ago, you can go back and perform a diff of the current state.


### Compliance and Auditing

Here are a few uses cases from PCI-DSS 3.2 and where security monkey comes handy:

| PCI # | PCI-DSS 3.2 | Security Monkey |
|-------|-------------|-----------------|
| 10.2.7 | Creation and Deletion of System level objects | Logs and Alerts on changes to: <ul><li>IAM Users</li><li>Security Groups</li><li>Elastic IP</li><li>Route 53</li></ul> |
| 10.5.4 | Write logs for external-facing technologies onto a secure, centralized, internal log or service media device | Logs for external-facing echnologies <ul><li>Security Group Config</li><li>ELB Config</li><li>SES</li><li>IAM are queried and monitored by Security</li></ul> |
| 10.6 | Review logs and security events for all system components to identify anomalies or suspicious activities | Security monkey examines security policies nd procedures to verify that the procedures are defined for reviewing items in **10.6.1** at least daily, either manually or via log tools |
| 10.6.1 | Review the following at least daily <ul><li>All security events</li><li>Logs of all components that store, process or transmit CHD and/or SAD</li><li>Logs all system critical components</li></ul> | Security Monkey sends daily reports on <ul><li>Security Groups</li><li>S3</li><li>IAM</li><li>ELB policies</li></ul> |
| 10.7 | Retain audit trail history for at least one year, with a minimum of three months immediately available for analysis | Security Monkey monitors the S3 bucket lifecycle, which shows the audit log retention policy |

Daily audit reports generated by Security Monkey

![Daily Reports in Google Mail](assets/daily_reports.png)


### How to set up Security Monkey?

##### Set up your IAM roles

At first, we are going to create two IAM roles namely `SecurityMonkeyInstaceProfile` and `SecurityMonkey`. Before creating these roles we need to create our own custom policies for them. Navigate to your [IAM console](https://console.aws.amazon.com/iam/) and on the left pane click on Policies. Choose to **Create a Policy** and select the JSON tab.

Paste the following JSON code in the given area:

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": [
      "ses:SendEmail"
    ],
    "Resource": "*"
  }, {
    "Effect": "Allow",
    "Action": "sts:AssumeRole",
    "Resource": "*"
  }]
}
```

After that name the policy `SecurityMonkeyLaunchPerms`. Similar to this create a new policy named `SecurityMonkeyReadOnly` with the following JSON code:

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Action": [
      "cloudwatch:Describe*",
      "cloudwatch:Get*",
      "cloudwatch:List*",
      "ec2:Describe*",
      "elasticloadbalancing:Describe*",
      "iam:List*",
      "iam:Get*",
      "route53:Get*",
      "route53:List*",
      "rds:Describe*",
      "s3:GetBucketAcl",
      "s3:GetBucketCORS",
      "s3:GetBucketLocation",
      "s3:GetBucketLogging",
      "s3:GetBucketPolicy",
      "s3:GetBucketVersioning",
      "s3:GetLifecycleConfiguration",
      "s3:ListAllMyBuckets",
      "sdb:GetAttributes",
      "sdb:List*",
      "sdb:Select*",
      "ses:Get*",
      "ses:List*",
      "sns:Get*",
      "sns:List*",
      "sqs:GetQueueAttributes",
      "sqs:ListQueues",
      "sqs:ReceiveMessage"
    ],
    "Effect": "Allow",
    "Resource": "*"
  }]
}
```

Now we are ready with our policies and we can proceed to our role creation. Navigate to your IAM console again and select **Roles** from the left pane. Click on **Create Role** and on the next page select **EC2** as the service.

![](assets/create_role.png)

In the next page add your previously created Policy `SecurityMonkeyLaunchPerms` to the role.

![](assets/attach_perms.png)

And on the last page, name your role `SecurityMonkeyInstaceProfile` and proceed with the role creation. Repeat the same steps to create another role and attach the Policy `SecurityMonkeyReadOnly` this time. Name this role `SecurityMonkey` and proceed with role creation.

![](assets/review.png)


After completing the above successfully, go to your newly created Role `SecurityMonkey` and navigate to the **Trust relationships** tab and add the following JSON code:

```json
{
  "Version": "2008-10-17",
  "Statement": [{
    "Sid": "",
    "Effect": "Allow",
    "Principal": {
      "AWS": [
        "arn:aws:iam::<YOUR ACCOUNT_ID GOES HERE>:role/SecurityMonkeyInstanceProfile"
      ]
    },
    "Action": "sts:AssumeRole"
  }]
}
```

> **NOTE:** You will have to add your own Account ID in the above code.

Your `SecurityMonkey` role should now look something like this:

![](assets/summary.png)

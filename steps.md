ACCOUNT A [SOURCE ACCOUNT]

Created the following AWS Identity Access Management (IAM) policy to provide access permissions to the S3 bucket: Lets call it “s3_policy_account_a”
```json
{
    "Version": "2012-10-17",
    "Statement": [{
            "Action": [
                "s3:ListBucket"
            ],
            "Effect": "Allow",
            "Resource": [
                "arn:aws:s3:::<s3_bucket_name_Account_A>"
            ]
        },
        {
            "Action": [
                "s3:GetObject",
                "s3:PutObject",
                "s3:DeleteObject",
                "iam:PassRole"
            ],
            "Effect": "Allow",
            "Resource": [
                "arn:aws:s3:::<s3_bucket_name_Account_A>/*"
            ]
        }
    ]
}
```
Create a role with the same policy named “s3_role_account_a” using the below trust relationship
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "",
            "Effect": "Allow",
            "Principal": {
                "Service": "opensearchservice.amazonaws.com"
            },
            "Action": "sts:AssumeRole"
        }
    ]
}
```
Created the following AWS Identity Access Management (IAM) policy to provide access permissions to perform the passrole action for the role created in step 2. Lets call it passrole_policy_account_a
```json
{
    "Version": "2012-10-17",
    "Statement": [{
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": "iam:PassRole",
            "Resource": "arn_of_s3_role_account_a"
        },
        {
            "Sid": "VisualEditor1",
            "Effect": "Allow",
            "Action": "es:ESHttpPut",
            "Resource": "arn_of_opensearch_domain_in_account_a/*"
        }
    ]
}
```
Create the role using the above policy and lets name it passrole_role_account_a. Use the following trust relationship. After creating the role, make sure you add AWSLambdaBasicExecutionRole and AWSLambdaVPCAccessExecutionRole managed policies to the same role.
```json
{
    "Version": "2012-10-17",
    "Statement": [{
            "Sid": "Statement1",
            "Effect": "Allow",
            "Principal": {
                "Service": [
                    "lambda.amazonaws.com"
                ]
            },
            "Action": "sts:AssumeRole"
        }

    ]
}
```

1. Download the zip file from my Github repo - https://github.com/Its-Ankush/opensearch_snapshot
2. Unzip the zip file and there will be a folder called Archive. Zip the contents inside the Archive folder [not the folder but the contents inside the folder] into a new folder called “lambda.zip”.
3. Head over to the lambda console, create a lambda function with Python 3.9, Execution role as passrole_role_account_a, VPC as the same VPC as the opensearch domain and for security group, you can use the same security group as opensearch domain but there needs to be a self reference to the security group for port 443 so that lambda can connect to it.
4. After lambda function gets created, make sure to change the Handler into "snapshot.lambda_handler" under Code > Runtime Settings > Edit > Handler. Make sure to change the timeout to 30 seconds under Configurations tab. 
5. Click on import from zip and select the zip “lambda.zip” and upload it.
6. Click on the snapshot.py file and change the role_arn in line 20 to the s3_role_account_a ARN. This code will register a manual snapshot repository.
7. To take the snapshot, just replace path value in line 8 to `_snapshot/<name_of_repository>/<snapshot_name>`


ACCOUNT B [ DESTINATION ACCOUNT]


Created the following AWS Identity Access Management (IAM) policy to provide access permissions to the S3 bucket: Lets call it “s3_policy_account_b”
```json
{
    "Version": "2012-10-17",
    "Statement": [{
            "Action": [
                "s3:ListBucket"
            ],
            "Effect": "Allow",
            "Resource": [
                "arn:aws:s3:::<s3_bucket_name_Account_A>"
            ]
        },
        {
            "Action": [
                "s3:GetObject",
                "s3:PutObject",
                "s3:DeleteObject",
                "iam:PassRole"
            ],
            "Effect": "Allow",
            "Resource": [
                "arn:aws:s3:::<s3_bucket_name_Account_A>/*"
            ]
        }
    ]
}
```
Create a role with the same policy named “s3_role_account_b” using the below trust relationship
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "",
            "Effect": "Allow",
            "Principal": {
                "Service": "opensearchservice.amazonaws.com"
            },
            "Action": "sts:AssumeRole"
        }
    ]
}
```
Created the following AWS Identity Access Management (IAM) policy to provide access permissions to perform the passrole action for the role created in step 2. Lets call it passrole_policy_account_b
```json
{
    "Version": "2012-10-17",
    "Statement": [{
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": "iam:PassRole",
            "Resource": "arn_of_s3_role_account_b"
        },
        {
            "Sid": "VisualEditor1",
            "Effect": "Allow",
            "Action": "es:ESHttpPut",
            "Resource": "arn_of_opensearch_domain_in_account_b/*"
        }
    ]
}
```
Create the role using the above policy and lets name it passrole_role_account_b. Use the following trust relationship. After creating the role, make sure you add AWSLambdaBasicExecutionRole and AWSLambdaVPCAccessExecutionRole managed policies to the same role.
```json
{
    "Version": "2012-10-17",
    "Statement": [{
            "Sid": "Statement1",
            "Effect": "Allow",
            "Principal": {
                "Service": [
                    "lambda.amazonaws.com"
                ]
            },
            "Action": "sts:AssumeRole"
        }

    ]
}
```

Switch to Account A’s S3 bucket and use the below bucket policy 
```json
{
    "Version": "2012-10-17",
    "Id": "Policy1568001010746",
    "Statement": [{
            "Sid": "Stmt1568000712531",
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn_of_s3_role_account_b"
            },
            "Action": "s3:*",
            "Resource": "arn:aws:s3:::<bucket_name_account_A>"
        },
        {
            "Sid": "Stmt1568001007239",
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn_of_s3_role_account_b"
            },
            "Action": "s3:*",
            "Resource": "arn:aws:s3:::<bucket_name_account_A>/*"
        }
    ]
}
```

Follow the same steps from Step 5  under SOURCE ACCOUNT except this time, the lambda function will be created in the VPC of the destination domain with the execution role as passrole_role_account_b.


1. Change the path in line 8 to the same path which is used for Source account [the path must have the repository with the same name]
2. Change the payload into the following 

```
payload = {
    "type": "s3",
    "settings": {
        "bucket": "<bucket_name>",
        "region": "us-east-1",
        "role_arn": "<s3_role_account_b>",
        "canned_acl": "bucket-owner-full-control"

    }
}
```
1. Execute the lambda code to register the repo with the same name
2. To restore the snapshot, use the path as 

`path = '_snapshot/<repo_name>/<snapshot_name>/_restore'`

1. Change the method type in line 23 to 

`r = requests.post(url, auth=awsauth, headers=headers);`

Execute the lambda code to restore the snapshot

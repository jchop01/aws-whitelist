# Overview
This AWS CloudFormation template is used to create a new user, allowing access strictly from a whitelisted CIDR Block of IP addresses.
The template takes the desired CIDR Block as input, and outputs the newly created users' ACCESS_KEY_ID and SECRET_ACCESS_KEY. The user can use these credentials to access ALL AWS services, when using the AWS CLI from an IP address in the whitelisted block.  The user will be denied access when attempting to use the credentials from an IP address outside of the whitelisted block.

# Running

## Prerequisties

- AWS Account - Get one [here](https://aws.amazon.com/console/)
- AWS CLI Installed - Instructions [here](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html)
- AWS CLI Configured with root credentials - Instructions [here](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html#post-install-configure)

## Deploy the Cloud Formation Stack
You can deploy the script with AWS CLI.

This example shows how to create a user with a whitelisted address of 1.1.1.2.

Run:
```
git clone https://github.com/jchop01/aws-whitelist.git
cd aws-whitelist
aws cloudformation deploy --template-file whitelist_user.yaml --stack-name test --parameter-overrides WhitelistedCIDR=1.1.1.2/32 --capabilities CAPABILITY_IAM
```

After creation, retreive the credentials of the new user.

Run:
```
aws cloudformation  describe-stacks --stack-name test
```
You should see JSON output which describes the stack.  You'll see the newly created credentials in the Outputs section.
```
{
    "Stacks": [
        {
            ....
            "Outputs": [
                {
                    "Description": "AWSSecretAccessKey of new user",
                    "OutputKey": "SecretKey",
                    "OutputValue": "****"
                },
                {
                    "Description": "AWSAccessKeyId of new user",
                    "OutputKey": "AccessKey",
                    "OutputValue": "****"
                }
            ],
            ....
        }
    ]
}
```

## Verification
We can run a simply command to test access using the newly created credentials.

On the machine that was whitelisted, run:
```
AWS_ACCESS_KEY_ID=<AWSAccessKeyId of new user>  AWS_SECRET_ACCESS_KEY=<AWSSecretAccessKey of new user> aws s3 ls
```
You should see a list of all S3 buckets, such as:
```
2019-09-23 18:03:50 cf-templates-*****-us-west-2
```

Change your IP address if possible, for example by connecting to a VPN.  Alternatively, run the same command from a different machine IP.  You should see an error indicating the error was denied.

```
An error occurred (AccessDenied) when calling the ListBuckets operation: Access Denied

```
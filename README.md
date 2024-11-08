# T(erra)F(orm)manual
Collections of terraform manuals

This manual will guide the reader through the process, starting from assuming AWS credentials, importing the Terraform configuration from AWS, generating generated.tf files, and re-format the outputs from generated.tf files to a format that follows conventions. Please mind that the explanation is made as simple as possible, to allow easier learning curve, this way, even interns to perform these tasks. Remember that any added complexity will increase error rate.

We are going to focus more on the scenario for re-generating terraform from current actual state. Mainly by creating new folder, new tfstate, importing necessary resources, and then re-format generated tf files as per the conventions.

## Preparation
**Step 1 : Preparing your imports:**
First you must locate the backend of your old terraform, or if you didn’t have terraform in the first place you must create a new backend. This backend will contain tfstate, which will be used to create a change plan whenever you update your terraform code.
Let's say we create this file by the name *terraform.tf*

```
provider "aws" {
  region = "ap-southeast-1"
}

terraform {
  backend "s3" {
    bucket         = "default-terraform-state-ap-southeast-1-accountnumber"
    key            = "ap-southeast-1/csatrio/csatrio-test/terraform.tfstate"
    region         = "ap-southeast-1"
    dynamodb_table = "default-terraform-state-ap-southeast-1-accountnumber"
  }
}
```
change the key to point to another folder in s3
```
key            = "ap-southeast-1/csatrio/csatrio-tests/terraform.tfstate"
```
This way you will end up with fresh terraform state.

After that you need to create import.tf describing which resource you want to import, for example importing an instance will be using this kind of syntax
```
import {
  to = aws_instance.instancetest
  id = "i-0f4a6ad9dfcf5b545"
}
```
importing security group is quite straightforward
```
import {
  to = aws_security_group.ec2_sg
  id = "sg-09f65bdfd89c4e9f4"
}
import {
  to = aws_security_group.elb_sg
  id = "sg-0941d4d659abaf3da"
}
```
Importing route 53
```
import {
  to = aws_route53_record.instancetest
  id = "Z3GYG4CSKWFHT7_instancetest.development.com_A"
}
```
Importing Load balancer by it's ARN
```
import {
  to = aws_lb.instancetest
  id = "arn:aws:elasticloadbalancing:ap-southeast-1:accountnumber:loadbalancer/app/yourlb/1b31a18d6112b922"
}
```
Create hidden file named `.terraform.version` and content 1.8.0 or greater
```
1.8.0
```

**Step2: Now that our preparation is finished, we can move on to the AWS:**
We assume that you already install the [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html).
Login to aws, part after the `--profile` had to be adjusted to your account
```
aws sso login --profile rolename@awsaccountname
eval "$(aws configure export-credentials --profile your-profile-name --format env)
# alternatively if it doesn't work
export AWS_ACCESS_KEY_ID=$(aws configure get $AWS_PROFILE.aws_access_key_id)
export AWS_SECRET_ACCESS_KEY=$(aws configure get $AWS_PROFILE.aws_secret_access_key)
export AWS_SESSION_TOKEN=$(aws configure get $AWS_PROFILE.aws_session_token)
```
then confirm and continue, you had no option anyway.
![image](https://github.com/user-attachments/assets/5b7fbe6b-4622-4081-9ecc-59f9c59dbd75)

**Step 3 : Terraform Init:**
after being able to login via browser and get the assume role in console, then do 
```
terraform init
# for mac laptops use this, for windows change the -platform=thing
terraform providers lock -platform=darwin_amd64 -platform=linux_amd64
```
if you encounter error on the step, you may check your aws config, possibly at `/Users/[user]/.aws/config`

## Generate resource
To import resources to file named generated.tf, we can execute the following command
```
terraform plan -generate-config-out=generated_resources.tf
```
given that these files were present and correctly configured:
- .terraform.version
- terraform.tf that contain location information to store the terraform.tfstate
- import.tf which contains information about what we are going to import

The generated.tf will look like this
<img width="1551" alt="image" src="https://github.com/user-attachments/assets/3f468626-ac11-480e-9fb6-5532abbd4ecc">
You will encounter these errors, because until now the config generation was experimental. Don’t be discouraged, just read the errors, and fix 1 by 1. _Tip: You can write a script to do common fix if you are given a task to import a large number of terraform, the script might save you from RSI (repetitive strain injury)._
![image](https://github.com/user-attachments/assets/3068a963-f30a-4406-a4da-f3fd396cb9ac)
### Testing whether we are successful
```terraform plan```
the plan should say that we import something, 0 add, 0 change, 0 destroy.
format your entire new folder using command
```teraform fmt```
After being sure that you won’t destroy anything, and check that everything is imported. You can finally execute 
```
terraform apply
terraform providers lock -platform=darwin_amd64 -platform=linux_amd64
git add .
git commit -S -m 'your_message'
git push origin <your-branch>
```
please note that the platform must be adjusted to your system platform and the server platform.
Good luck with the terraform.

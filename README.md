# T(erra)F(orm)manual
Collections of terraform manuals

This manual will guide the reader through the process, starting from assuming AWS credentials, importing the Terraform configuration from AWS, generating generated.tf files, and re-format the outputs from generated.tf files to a format that follows conventions. Please mind that the explanation is made as simple as possible, to allow easier learning curve, this way, even interns to perform these tasks. Remember that any added complexity will increase error rate.

We are going to focus more on the scenario for re-generating terraform from current actual state. Mainly by creating new folder, new tfstate, importing necessary resources, and then re-format generated tf files as per the conventions.

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


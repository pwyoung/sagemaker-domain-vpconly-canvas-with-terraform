# GOAL

Document problem deploying AWS Sagemaker in us-east-1.

# TL/DR

There is a problem, of some sort, with AWS Sagemaker in us-east-1.
I got 5x failures trying to deploy to us-east-1.
It worked immediately with us-east-2.

# Details

Used the following code as-is
  https://github.com/aws-samples/sagemaker-domain-vpconly-canvas-with-terraform/tree/main

It should create AWS Sagemaker in a private VPC, similar to this:
  https://docs.aws.amazon.com/sagemaker/latest/dg/canvas-vpc.html#canvas-vpc-configure

# ERROR

While running "terraform apply", this error showed up.

module.sagemaker_domain.aws_sagemaker_user_profile.default_user: Creation complete after 4s [id=arn:aws:sagemaker:us-east-1:792566780889:user-profile/d-72kuqyzmpwim/defaultuser]
╷
│ Error: creating EC2 VPC Endpoint (com.amazonaws.us-east-1.comprehend): InvalidParameter: The VPC endpoint service com.amazonaws.us-east-1.comprehend does not support the availability zone of the subnet: subnet-0b49bdb25117da505.
│ 	status code: 400, request id: a467ee01-4167-453e-bacd-7145c05c2140
│
│   with module.sagemaker_domain.module.sagemaker_domain_vpc.aws_vpc_endpoint.interface_endpoints_canvas["com.amazonaws.us-east-1.comprehend"],
│   on ../submodules/vpc/vpc.tf line 188, in resource "aws_vpc_endpoint" "interface_endpoints_canvas":
│  188: resource "aws_vpc_endpoint" "interface_endpoints_canvas" {
│
╵

# ROOT-CAUSE

VPC Endpoing "com.amazonaws.us-east-1.comprehend" is not available in an AZ in us-east-1.

Details:
  Confirmed via the AWS Console:
    https://us-east-1.console.aws.amazon.com/vpcconsole/home?region=us-east-1#CreateVpcEndpoint:
    It is available in "us-east-1a(use1-az6)" BUT NOT in "us-east-1b(use1-az1)"

# WORK-AROUND

# Create a variable override file, e.g.
echo 'aws_region = "us-east-2"' > ./custom.auto.tfvars


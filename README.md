# 1. Configure VPC
```
aws cloudformation --region us-east-1 create-stack --stack-name my-vpc --template-body file://vpc.yaml
aws cloudformation describe-stacks --stack-name my-vpc
```

# 2. Create instance key

```
aws ec2 create-key-pair --region us-east-1 --key-name my-key --query "KeyMaterial" --output text > ~/.ssh/my-key.pem
chmod 600  ~/.ssh/my-key.pem
```

# 3. Deploy your LB and ASG

```
YOUR_EMAIL=<YOUR_EMAIL>
PUBLIC_SUBNETS=$(aws ec2 describe-subnets --filter 'Name=tag:Name,Values=my-public-*' --query 'Subnets[*].[SubnetId]' --output text | tr '\n' ',' | sed 's/.$//')

aws cloudformation --region us-east-1 create-stack --stack-name my-lb-app --template-body file://lb-asg.json --parameters \
ParameterKey=KeyName,ParameterValue=my-key \
ParameterKey=OperatorEMail,ParameterValue=$YOUR_EMAIL \
ParameterKey=Subnets,ParameterValue='"$PUBLIC_SUBNETS"' \
ParameterKey=VpcId,ParameterValue=$(aws ec2 describe-vpcs --filters 'Name=tag:Name,Values=my-vpc' --query 'Vpcs[*].VpcId' --output text) \
ParameterKey=SSHLocation,ParameterValue="$(curl checkip.amazonaws.com)/32"
```

# 4. Stress your server and monitor

```
sudo yum install stress
stress --cpu 8
```

# 5. Upload your results

Make a brief description of your results and take a screenshot from your ASG metrics dashboard and upload it to the platform.



# Credits to
[Base VPC Template](https://gist.github.com/lizrice/5889f33511aab739d873cb622688317e)
[Walkthrough: Create a scalable, load-balancing web server](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/example-templates-autoscaling.html)


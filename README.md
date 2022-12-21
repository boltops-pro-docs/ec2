<!-- note marker start -->
**NOTE**: This repo contains only the documentation for the private BoltsOps Pro repo code.
Original file: https://github.com/boltopspro/ec2/blob/master/README.md
The docs are publish so they are available for interested customers.
For access to the source code, you must be a paying BoltOps Pro subscriber.
If are interested, you can contact us at contact@boltops.com or https://www.boltops.com

<!-- note marker end -->

# EC2 CloudFormation Blueprint

[![Watch the video](https://img.boltops.com/boltopspro/video-preview/blueprints/ec2.png)](https://www.youtube.com/watch?v=UxDlPfR9jhQ)

![CodeBuild](https://codebuild.us-west-2.amazonaws.com/badges?uuid=eyJlbmNyeXB0ZWREYXRhIjoiTEVUckZ3a3g0SWtVaGVnY3F2M1JGNzZqL2V0UDl2aDNmZ25OMjU4WW1nVmtHekVuL1BDNHRDTEpESWZKaHlHUTJlWnlqZWxiY1N5eWVvU05XdEp1S29rPSIsIml2UGFyYW1ldGVyU3BlYyI6InlEK1dQOEFEUVdYUzV2ekMiLCJtYXRlcmlhbFNldFNlcmlhbCI6MX0%3D&branch=master)

[![BoltOps Badge](https://img.boltops.com/boltops/badges/boltops-badge.png)](https://www.boltops.com)

This blueprint provisions an EC2 instance.  This blueprint is useful if you just need a single server. Examples include Jenkins, Wordpress, and more.

* Several [AWS::EC2::Instance](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-ec2-instance.html) properties are configurable with [Parameters](https://lono.cloud/docs/configs/params/). Additionally, properties that require further customization are configurable with [Variables](https://lono.cloud/docs/configs/shared-variables/).  The blueprint is extremely flexible and configurable for your needs.
* You can launch the instance in a Custom VPC and Subnet by configuring `VpcId` and `SubnetId`.
* You can customize the UserData script and control the bootstrap process with a `@user_data_script` variable.
* You can assign existing Security Groups to the instance or have the blueprint create a managed Security Group.
* You can optionally create a [Route53 Record](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-route53-recordset.html) and point it to the EC2 dns name.
* You can optionally create an [EIP](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-ec2-eip.html) associated with the EC2 instance with the `CreateEip` parameter.

## Usage

1. Add blueprint to Gemfile
2. Configure: configs/ec2 values
3. Deploy

## Add

Add the blueprint to your lono project's `Gemfile`.

```ruby
gem "ec2", git: "git@github.com:boltopspro/ec2.git"
```

## Configure

First you want to configure the [configs](https://lono.cloud/docs/core/configs/) files. Use [lono seed](https://lono.cloud/reference/lono-seed/) to configure starter values quickly.

    LONO_ENV=development lono seed ec2

To deploy to additional environments:

    LONO_ENV=production  lono seed ec2

The generated files in `config/ec2` folder look something like this:

    configs/ec2/
    ├── params
    │   ├── development.txt
    │   └── production.txt
    └── variables
        ├── development.rb
        └── production.rb

## Deploy

Use the [lono cfn deploy](http://lono.cloud/reference/lono-cfn-deploy/) command to deploy. Example:

    LONO_ENV=development lono cfn deploy ec2-development --blueprint ec2 --sure
    LONO_ENV=production  lono cfn deploy ec2-production  --blueprint ec2 --sure

## Configure: More Details

### Custom UserData Script

The UserData can be customized with the `@user_data_script` variable.  The variable should be set to the path of the script. Example:

configs/ec2/variables/development.rb:

```ruby
@user_data_script = "configs/ec2/user_data/bootstrap.sh"
```

The script is wrapped in a [base64](https://lono.cloud/docs/intrinsic-functions/base64/) and [sub](https://lono.cloud/docs/intrinsic-functions/sub/) call. So [Pseudo Parameters](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/pseudo-parameter-reference.html) are available to be used in the script if needed. Example:

configs/ec2/user_data/bootstrap.sh

    echo ${AWS::StackName}

The custom `@user_data_script` is appended to an existing default UserData script that ships with the blueprint. The UserData runs cfn-init and applies configsets before the custom `@user_data_script`.

### Stack Name Convention

By leveraging the lono Stack Name and [CLI conventions](https://lono.cloud/docs/conventions/cli/), we can organize the configs files in a way that matches the stack name. Example:

    lono cfn deploy daisy   --blueprint ec2
    lono cfn deploy jenkins --blueprint ec2

Will use the corresponding config files:

    configs/ec2/development/daisy.txt
    configs/ec2/development/jenkins.txt

### Custom VPC

To provision the EC2 instance to a custom vpc, provide the `SubnetId` and `VpcId` parameter. The `SubnetId` is used for [AWS::EC2::Instance](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-ec2-instance.html) resource and the `VpcId` is used for the [AWS::EC2::SecurityGroup](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-ec2-security-group.html). Example:

    SubnetId=subnet-111
    VpcId=vpc-111

### Security Groups

To assign existing security groups to the EC2 instance use `SecurityGroupIds`. Example:

    SecurityGroupIds=sg-111,sg-222

If not set, then the blueprint will create a managed Security Group and assign to it to the EC2 instance.

### Managed Security Group Rules

To open security group rules on the Managed Security Group you can use the `@security_group_ingress` variable. Example:

configs/ec2/variables/development.rb:

```ruby
@security_group_ingress = [{
  CidrIp: "0.0.0.0/0",
  FromPort: 22,
  IpProtocol: "tcp",
  ToPort: 22,
}]
```

### Larger Root Volume Size

To specify a larger root volume size for the EC2 instance, use the `@block_device_mappings` variable. Example:

configs/ec2/variables/development.rb:

```ruby
@block_device_mappings = [
  DeviceName: "/dev/xvda",
  Ebs: {
    VolumeSize: 30
  }
]
```

### Route53 DNS Pretty Host Name

You can use `HostedZoneId` or `HostedZoneName` to create a pretty endpoint pointing to the EC2 instance.  You can control whether the route53 record connects to the public or private DNS name of the instance with `ConnectToDns=public` or `ConnectToDns=private`.
Example:

    DnsName=my-instance.example.com.
    HostedZoneName=example.com.
    ConnectToDns=public

If you have configured `CreateEip=1` then the route53 record will point to the EIP instead.

### EIP

You can use `CreateEip=1` and the blueprint will create an EIP and associate it with the EC2 instance.

## Blueprint Configsets

This blueprint includes the following [blueprint configsets](https://lono.cloud/docs/configsets/blueprint/):

* [awslogs](https://github.com/boltopspro-docs/awslogs): Centralized logging of the Instance logs to CloudWatch Logs.
* [cfn-hup](https://github.com/boltopspro-docs/cfn-hup): Continuous configuration management to automatically update instance.
* [ssm](https://github.com/boltopspro-docs/ssm): Secure ssh and session manager access to the instance.

This means the instance is already set up with centralized logging, cfn-hup for continuously configuration management updates, and ssm for session manager secure access.

Refer to each configsets README on details for further customization.  For example, you can [customize what logs get sent](https://github.com/boltopspro-docs/awslogs#customize-awslogsconf) to CloudWatch logs.

## Project Configsets

You may want to add additional configsets. Examples:

* [httpd](https://github.com/boltopspro-docs/httpd): Httpd Apache2 Webserver
* [jenkins](https://github.com/boltopspro-docs/jenkins): Jenkins
* [ruby](https://github.com/boltopspro-docs/ruby): Ruby

To configure additional configsets. First, add them to the project Gemfile. Example:

Gemfile:

```ruby
gem "ruby", git: "git@github.com:boltopspro/ruby"
```

Then configure the configset in the `configs/ec2` folder.

configs/ec2/configsets/development.rb:

```ruby
configset("ruby", resource: "Instance")
```

You can verify that its added with the [lono configsets BLUEPRINT](https://lono.cloud/reference/lono-configsets/) command.  Example:

    $ lono configsets ec2
    Using configsets for development: configs/ec2/configsets/development.rb
    Configsets used by ec2 blueprint:
    +---------------------+----------------------------------------------------------------------------------+
    |        Name         |                     Path                              |     Type     |   From    |
    +---------------------+----------------------------------------------------------------------------------+
    | amazon-linux-extras | ..2.5.0/bundler/gems/amazon-linux-extras-531b03e88ef4 | materialized | project   |
    | ruby                | ..2.5.0/bundler/gems/ruby-fca48d80cf9b                | gem          | project   |
    | cfn-hup             | vendor/configsets/cfn-hup                             | vendor       | blueprint |
    | awslogs             | vendor/configsets/awslogs                             | vendor       | blueprint |
    | ssm                 | vendor/configsets/ssm                                 | vendor       | blueprint |
    +---------------------+----------------------------------------------------------------------------------+
    $

More info: [Project Configsets](https://lono.cloud/docs/configsets/project/)

## IAM Permissions

The IAM permissions required for this stack are described below.

Service | Description
--- | ---
cloudformation | To launch the CloudFormation stack.
ec2 | EC2 instance and security group.
route53 | Route53 pretty endpoint
s3 | Lono managed s3 bucket

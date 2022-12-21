<!-- note marker start -->
**NOTE**: This repo contains only the documentation for the private BoltsOps Pro repo code.
Original file: https://github.com/boltopspro/ec2/blob/master/CHANGELOG.md
The docs are publish so they are available for interested customers.
For access to the source code, you must be a paying BoltOps Pro subscriber.
If are interested, you can contact us at contact@boltops.com or https://www.boltops.com

<!-- note marker end -->

# Change Log

All notable changes to this project will be documented in this file.
This project *loosely tries* to adhere to [Semantic Versioning](http://semver.org/).

## [2.0.5]
- improve starter seed files

## [2.0.4]
- add lono_type to gemspec

## [2.0.3]
- update readme and starter seed

## [2.0.2]
- allow ImageId parameter override

## [2.0.1]
- improve SubnetId and VpcId comments

## [2.0.0]
- #3 upgrade to lono v7
- allow custom UserData with @user_data_script variable
- allow custom IAM permissions with @iam_managed_policy_arns and @iam_policies variables
- use parameter_group
- update AmazonLinux2 AMI
- use configsets: cfn-hup, awslogs, ssm
- allow custom security group ingress rules with variable @security_group_ingress variable

## [1.1.1]
- docs: Larger Volume Size
- use built-in `user_data_script` helper instead

## [1.1.0]
- Add EIP support

## [1.0.0]
- upgrade to lono v6
- add parameters for simple types and variables for complex types
- add route53 support
- add UserData customization with `@user_data_script` variable
- #2 lono v6 upgrade: auto_camelize false
- update amis: linuxamazon2

## [0.2.0]
- #1 upgrade lono v5.2
- use camelize naming

## [0.1.0]
- Initial release

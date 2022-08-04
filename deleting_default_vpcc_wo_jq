#!/usr/bin/env bash

# based on https://gist.github.com/jokeru/e4a25bbd95080cfd00edf1fa67b06996
# Made modifications to remove need for jq, set AWS_DEFAULT_REGION so you don't have to specify --region explicitly
#  Add region 'human-readable' names
REGIONS='us-east-1 (N. Virginia)
us-east-2 (Ohio)
us-west-1 (California)
us-west-2 (Oregon)
eu-central-1 (Frankfurt)
eu-west-1 (Ireland)
eu-west-2 (London)
eu-west-3 (Paris)
eu-north-1 (Stockholm)
ap-northeast-1 (Tokyo)
ap-northeast-2 (Seoul)
ap-south-1 (Mumbai)
ap-southeast-1 (Singapore)
ap-southeast-2 (Sydney)
ca-central-1 (Toronto)
sa-east-1 (Sao Paolo)
ap-northeast-3 (Osaka)
ap-east-1 (Honk Kong)
me-south-1 (Bahrain)
af-south-1 (Cape Town)
eu-south-1 (Milan)'

INDENT='    '

echo "Using profile $AWS_PROFILE"

aws ec2 describe-regions --output text --query 'Regions[].[RegionName, OptInStatus]' | sort -r \
| while read REGION OptInStatus; do

  export AWS_DEFAULT_REGION=$REGION
  RegName=$( echo "$REGIONS" | grep "^${REGION}" )
  [ -z "$RegName" ] && RegName="$REGION"
  echo "* Region ${RegName}"

  # get default vpc
  vpc=$( aws ec2  describe-vpcs --filter Name=isDefault,Values=true --output text --query 'Vpcs[0].VpcId' )
  if [ "${vpc}" = "None" ]; then
    echo "${INDENT}No default vpc found"
    continue
  fi
  echo "${INDENT}Found default vpc ${vpc}"

  # get internet gateway
  igw=$(aws ec2 describe-internet-gateways --filter Name=attachment.vpc-id,Values=${vpc}  --output text --query 'InternetGateways[0].InternetGatewayId' )
  if [ "${igw}" != "None" ]; then
    echo "${INDENT}Detaching and deleting internet gateway ${igw}"
    aws ec2 detach-internet-gateway --internet-gateway-id ${igw} --vpc-id ${vpc}
    aws ec2 delete-internet-gateway --internet-gateway-id ${igw}
  fi

  # get subnets
  subnets=$(aws ec2 describe-subnets --filters Name=vpc-id,Values=${vpc} --output text --query 'Subnets[].SubnetId' )
  if [ "${subnets}" != "None" ]; then
    for subnet in ${subnets}; do
      echo "${INDENT}Deleting subnet ${subnet}"
      aws ec2 delete-subnet --subnet-id ${subnet}
    done
  fi

  # https://docs.aws.amazon.com/cli/latest/reference/ec2/delete-vpc.html
  # - You can't delete the main route table
  # - You can't delete the default network acl
  # - You can't delete the default security group

  # delete default vpc
  echo "${INDENT}Deleting vpc ${vpc}"
  aws ec2 delete-vpc --vpc-id ${vpc}
done

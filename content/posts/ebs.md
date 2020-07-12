+++
categories = []
date = ""
draft = true
meta = false
tags = []
title = ""

+++
# EBS device mapping on non Amazon Linux instances

Amazon Machine Images (AMI) cme

If you attach an EBS volume to an Ubuntu 18.04 instance (HVM), you'll find out that the device path shown by AWS API/UI doesn't match the device path in the instance. Ths is something exclusive for Ubuntu, but for AMIs not create by Amazon.

I have create

I have a test `t2.micro` instance named `ebs-test-ubuntu-18.04`. Let's get the instance ID:

```bash
$ aws ec2 describe-instances --filters "Name=tag:Name,Values=ebs-test-ubuntu-18.04" --query "Reservations[].Instances[].InstanceId"
[
    "i-0c09a10921d29977b"
]
```

```bash
# list volumes attached to instance
$ aws ec2 describe-volumes \
    --region us-east-1 \
    --filters Name=attachment.instance-id,Values=i-1234567890abcdef0 Name=attachment.delete-on-termination,Values=true
```

it's not available the path suggseted by amazon.
TODO - show how to check this using the AWS CLI.

This supposes a problem if we are automating using a configuration management tool as Ansible and it queries the required information from the instance MetaData.  
The problem gets even worse as this may vary depending of the type of virtualization used by the EC2 instance:

# ways of listing the device 

## xen intances

## nitro instances

## both types of instances


https://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/nvme-ebs-volumes.html#identify-nvme-ebs-device")

udev rule  
https://asanga-pradeep.blogspot.com/2018/12/udev-rules-for-aws-ebs-volumes.html

## IDEA - Create a diagram showing which instances use Xen and which ones use Nitro 

## How does this affect Nitro instances?  
If your instance uses [Nitro][https://aws.amazon.com/ec2/nitro/], here is how to check the path:  
The Nitro naming scheme is:  

```bash
/dev/nvme[0-26]n1/
```

```bash
root@ip-172-31-83-135:~# lsblk -o NAME,SERIAL | grep volnvme0n1 vol01ff51f17da446c63nvme1n1 vol094f64274fd381d7bnvme2n1 vol0e67261aa75b344c2
```

### How to solve it on Nitro  
devices follow the /dev/nvme\[0-26\]n1 naming convention (/dev/nvme0n1, /dev/nvme1n1, …), instead of the usual naming such as /dev/xvda or /dev/sdf.  

## How does this affect Xen instances  
The Xen naming convention is as follows:  

```bash
/dev/xvd\[1-N\]
```
If your instances use xen, here is how to check the path:  
All these tests where performed using GP2 volumes.  

## EBS disks on ubuntu  

The SERIALS match the volume ID on amazon UI  
root@ip-172-20-0-122:/dev/disk# blkid/dev/nvme0n1: UUID="2c195d14-ea1f-4786-8ccc-41544855f5af" TYPE="xfs"/dev/nvme1n1p1: LABEL="cloudimg-rootfs" UUID="3636f5d4-1dd0-4284-b5af-9b234b433392" TYPE="ext4" PARTUUID="e205f4c6-01"/dev/loop0: TYPE="squashfs"/dev/loop1: TYPE="squashfs"/dev/loop2: TYPE="squashfs"/dev/loop3: TYPE="squashfs"/dev/loop4: UUID="20bf3dda-9afc-4a5f-b812-2119160378f0" UUID_SUB="ebbb7cd4-4440-421a-8b13-d2537001cceb" TYPE="btrfs"/dev/nvme1n1: PTUUID="e205f4c6" PTTYPE="dos"

root@ip-172-20-0-122:/dev/disk# lsblk -fNAME FSTYPE LABEL UUID MOUNTPOINTloop0 squashfs /snap/core/8213loop1 squashfs /snap/core/8268loop2 squashfs /snap/amazon-ssm-agent/1455loop3 squashfs /snap/amazon-ssm-agent/1480loop4 btrfs 20bf3dda-9afc-4a5f-b812-2119160378f0 /data/concourse/workdir/volumesnvme0n1 xfs 2c195d14-ea1f-4786-8ccc-41544855f5af /datanvme1n1└─nvme1n1p1 ext4 cloudimg-rootfs 3636f5d4-1dd0-4284-b5af-9b234b433392 /  
METAURL="http://169.254.169.254/2012-01-12/meta-data/block-device-mapping/"for bd in \`curl -s $METAURL\`; do curl -s $METAURL$bd | \\ perl -pe 'BEGIN { $d = shift } s/^(\\/dev\\/)?(sd)?(.*)$/\\/dev\\/xvd$3 is $d\\n/' $bd; \\ done | sort  
Can I write a module for this?  

https://www.laurentgodet.com/2019/10/ebs-nvme-block-device-mapping-using-volume-ids/

 rewrite this basic networking concepts but using terraform


## References:

Brendan Greg on Nitro: http://www.brendangregg.com/blog/2017-11-29/aws-ec2-virtualization-2017.html
Nitro instances - https://aws.amazon.com/ec2/nitro/
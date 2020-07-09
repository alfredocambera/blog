+++
categories = []
date = ""
draft = true
meta = false
tags = []
title = ""

+++
\# EBS disk volume mapping in Ubuntu

If you have had to work mounting EBS instances using AMI other than \`Amazon Linux AMI\` you may have found that one you have attached a new volume, it's path is not available in the path suggeted by amazon (TODO - show how to check this using the AWS CLI).`Tis supposes as`roblem if we are automating using a configuration management tool as Ansible and it queries the required information from the instance MetaData.  
The problem gets even worse as this may vary depending of the type of virtualization used by the EC2 instance:

[https://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/nvme-ebs-volumes.html#identify-nvme-ebs-device](https://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/nvme-ebs-volumes.html#identify-nvme-ebs-device "https://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/nvme-ebs-volumes.html#identify-nvme-ebs-device")

IDEA - Create a diagram showing which instances use Xen and which ones use Nitro  
\## How does this affect Nitro instances?  
If your instance uses \[Nitro\]([https://aws.amazon.com/ec2/nitro/](https://aws.amazon.com/ec2/nitro/ "https://aws.amazon.com/ec2/nitro/")), here is how to check the path:  
The Nitro naming scheme is:  
\`\`\`bash/dev/nvme\[0-26\]n1 \`\`\`  
\`\`\`bashroot@ip-172-31-83-135:\~# lsblk -o NAME,SERIAL | grep volnvme0n1 vol01ff51f17da446c63nvme1n1 vol094f64274fd381d7bnvme2n1 vol0e67261aa75b344c2\`\`\`  
\### How to solve it on Nitro  
devices follow the /dev/nvme\[0-26\]n1 naming convention (/dev/nvme0n1, /dev/nvme1n1, …), instead of the usual naming such as /dev/xvda or /dev/sdf.  
\## How does this affect Xen instances  
The Xen naming convention is as follows:  
\`\`\`/dev/xvd\[1-N\]\`\`\`  
If your instances use Xen, here is how to check the path:  
All these tests where performed using GP2 volumes.  
\## EBS disks on ubuntu  
The SERIALS match the volume ID on amazon UI  
root@ip-172-20-0-122:/dev/disk# blkid/dev/nvme0n1: UUID="2c195d14-ea1f-4786-8ccc-41544855f5af" TYPE="xfs"/dev/nvme1n1p1: LABEL="cloudimg-rootfs" UUID="3636f5d4-1dd0-4284-b5af-9b234b433392" TYPE="ext4" PARTUUID="e205f4c6-01"/dev/loop0: TYPE="squashfs"/dev/loop1: TYPE="squashfs"/dev/loop2: TYPE="squashfs"/dev/loop3: TYPE="squashfs"/dev/loop4: UUID="20bf3dda-9afc-4a5f-b812-2119160378f0" UUID_SUB="ebbb7cd4-4440-421a-8b13-d2537001cceb" TYPE="btrfs"/dev/nvme1n1: PTUUID="e205f4c6" PTTYPE="dos"

root@ip-172-20-0-122:/dev/disk# lsblk -fNAME FSTYPE LABEL UUID MOUNTPOINTloop0 squashfs /snap/core/8213loop1 squashfs /snap/core/8268loop2 squashfs /snap/amazon-ssm-agent/1455loop3 squashfs /snap/amazon-ssm-agent/1480loop4 btrfs 20bf3dda-9afc-4a5f-b812-2119160378f0 /data/concourse/workdir/volumesnvme0n1 xfs 2c195d14-ea1f-4786-8ccc-41544855f5af /datanvme1n1└─nvme1n1p1 ext4 cloudimg-rootfs 3636f5d4-1dd0-4284-b5af-9b234b433392 /  
METAURL="http://169.254.169.254/2012-01-12/meta-data/block-device-mapping/"for bd in \`curl -s $METAURL\`; do curl -s $METAURL$bd | \\ perl -pe 'BEGIN { $d = shift } s/^(\\/dev\\/)?(sd)?(.*)$/\\/dev\\/xvd$3 is $d\\n/' $bd; \\ done | sort  
Can I write a module for this?  
[https://www.laurentgodet.com/2019/10/ebs-nvme-block-device-mapping-using-volume-ids/](https://www.laurentgodet.com/2019/10/ebs-nvme-block-device-mapping-using-volume-ids/ "https://www.laurentgodet.com/2019/10/ebs-nvme-block-device-mapping-using-volume-ids/")  
\# rewrite this basic networking concepts but using terraformhttps://grahamlyons.com/article/everything-you-need-to-know-about-networking-on-aws  
TODO and questions:  
\- Does this work on CentOS?
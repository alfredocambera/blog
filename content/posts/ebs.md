+++
categories = []
date = ""
draft = true
meta = false
tags = []
title = ""

+++
# # Fixing EBS incorrect device mapping on EC2 instances

\**Pending**

\* Add graphic of Nitro, Non-Nitro instances.

\* Name the type of instance that I used 

\* Explain that we are going to work with two types of inst# Fixing EBS incorrect device mapping on EC2 instances



**Pending**

* Add graphic of Nitro, Non-Nitro instances.
* Name the type of instance that I used 
* Explain that we are going to work with two types of instances (a graph?)
* Set a clear hostname for each instance type
* Add a conclusion
* Improve references
* Give a better explanation about how to use nitro https://aws.amazon.com/ec2/nitro/
* Show more that one way to get the vol-serial
* Explain about para virtual vs HVM
* What is NVMe?
* ==>>Amazon recomnends to do volume mapping using volume ID (`fstab`)
* Check if un-plugging and plugging a device again will affect its device path
* I set up the aws cli locally on each box (link to configuration)
* I need a better name for non nitro instances
* cada instancia tiene insalado awscli y configurado para tener acceso al API
* Add the reference to the python script I made
* Uptade code to ubuntu 20.04
* /dev/xvd - xen virtual device
* ~~Creo que justo lo primera oración del post: "Amazon Machine Images (AMIs) come in many flavors." se queda cómo volando porque justo después de eso hablas de EBS y EC2. Podrías quitarla y creo que no afectaría en nada~~

Have you attached an EBS volume to an EC2 instance and found out that the path provided by AWS doesn't match the path used by the instance? This happens because it depends on the block device driver used the Linux kernel.

There are as many kernel versions as [AMI](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html)s. This doesn't mean that the naming scheme is completely different from AMI to AMI. It means that this variation exists, and you need to be aware of it so you can realiabily manage EBS volumes.

Something important to note is that this issue doesn't exists for [Amazon Linux AMIs]([https://aws.amazon.com/amazon-linux-ami/) users. These images are supported by Amazon and create the correct device paths for attached EBS volumes.


Another factor that affecs how device paths are generated is the virtualization technoogy used by the EC2 instance.



, The problem doesn't stop there. Depending of the virtualization technology used by the instance type that you choose, it may also change. 

> Gráfico en disco en el medio, luego el driver, luego como se ve, lo que consigo cuando veo el API
>
> Esto para una imagen linux y una image de ubuntu 18




> grafico anterior más

There is another factor that affects the naming: HVM, PVM



so if you are provisioning an EC2 instance using a tool that retrieves this information from the API to, let's say, format a disk it will throw an error message as the disk won't be in the required path.







![test](https://media.giphy.com/media/UoqIxMikuK1RQt4w1r/giphy-downsized.gif)
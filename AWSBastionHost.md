# AWS Bastion Host

How to connect from your local machine to a Linux machine in private subnet on AWS?

Setup:
- security group _public-sg_ with port 22 (ssh) open to (at least) your local machine's IP (source)
- EC2 host (e.g. t2.micro) _bastion_ with e.g. Ubuntu 16.04 with public IP and DNS _ec2-1-2-3-4.eu-central-1.compute.amazonaws.com_ and in security group _public-sg_
- security group _private-sg_ with port 22 (ssh) open to at least the jump host's private IP
- EC2 host _target_ with e.g. Ubuntu 16.04 with no public IP, only private IP and DNS _ip-5-6-7-8.eu-central-1.compute.internal_ and in security group _private-sg_
- both machines use the same public/private key pair _my-aws-key_
- PEM file is available on local machine

### Add private key to auth agent
```
$ ssh-agent bash

$ ssh-add my-aws-key.pem

$ ssh-add -L
ssh-rsa AAAA...XYZ1a my-aws-key.pem
```

### Change ssh config on your local machine
```
$ vi ~/.ssh/config

$ cat ~/.ssh/config
Host *.compute.internal
  ProxyJump ubuntu@ec2-1-2-3-4.eu-central-1.compute.amazonaws.com
```

### Logon to the target host "directly" from your lcoal machine
```
$ ssh ubuntu@ip-5-6-7-8.eu-central-1.compute.internal
_
Welcome to Ubuntu 16.04.4 LTS (GNU/Linux 4.4.0-1052-aws x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  Get cloud support with Ubuntu Advantage Cloud Guest:
    http://www.ubuntu.com/business/services/cloud

0 packages can be updated.
0 updates are security updates.


Last login: Thu Apr 11 11:11:00 2018 from 1.2.3.4
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

ubuntu@ip-5-6-7-8:~$
```

### References
https://heipei.github.io/2015/02/26/SSH-Agent-Forwarding-considered-harmful/
https://www.digitalocean.com/community/tutorials/how-to-configure-custom-connection-options-for-your-ssh-client
https://en.wikibooks.org/wiki/OpenSSH/Cookbook/Proxies_and_Jump_Hosts
https://aws.amazon.com/blogs/security/securely-connect-to-linux-instances-running-in-a-private-amazon-vpc/

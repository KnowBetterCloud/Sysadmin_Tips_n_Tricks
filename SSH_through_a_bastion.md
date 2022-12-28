# SSH through a bastion

A common use-pattern is to deploy a number of cloud resources in private subnets (i.e. no ingress from Internet).  As a result, many folks will deploy a "bastion node" in a public subnet which allows SSH from the Internet (or specific IP or ranges).  Then SSH to their private resources.  

## Challenge
As you likely know, the only allowed ssh connection to AWS EC2 instance is as the ec2-user using an key-pair that was selected during the node deployment.  To connect to your private node, you would need to copy the private key to the bastion node.  Or not...

NOTE: copying your private key to your bastion creates additional risk which you need to assess. If the bastion was to become compromised, that key could be then gathered and used.
Additionally, there are risks associated with using SSH Agent Fowarding.  If the bastion was comprimised the socket created for the Agent could be exploited.  Therefore, you should limit what IP(s) can access the bastion.  And you can also use the ssh option 'ProxyCommand' (I will detail this in a future post).

## Solution
You can use SSH Agent Forwarding (ssh -A) to forward your ssh key.  This means that you do not need to add your private key to the bastion.

## References
[openSSH man pages](https://www.openssh.com/manual.html)  
[ssh (client) man pages](https://man.openbsd.org/ssh)


## Example
In this example I have 2 nodes

| public IP     | Private IP |
|:-------------:|:----------:|
| 35.170.78.188 | 10.0.0.57  |
| N/A           | 10.0.1.212 | 


### Without using forwarding (FAIL)
I added some line spacing for clarity
```
mylaptop:~ myuser$ mv ~/Downloads/3tierTest.cer ~/.ssh

mylaptop:~ myuser$ chmod 0600 ~/.ssh/3tierTest.cer

mylaptop:~ myuser$ ssh -i ~/.ssh/3tierTest.cer 35.170.78.188 -l ec2-user
Last login: Tue Dec 27 21:51:13 2022 from redacted.example.com

       __|  __|_  )
       _|  (     /   Amazon Linux 2 AMI
      ___|\___|___|

https://aws.amazon.com/amazon-linux-2/
30 package(s) needed for security, out of 49 available
Run "sudo yum update" to apply all updates.

[ec2-user@ip-10-0-0-57 ~]$ hostname -i
fe80::f8:50ff:fe76:410d%eth0 10.0.0.57

[ec2-user@ip-10-0-0-57 ~]$ ssh 10.0.1.212
The authenticity of host '10.0.1.212 (10.0.1.212)' can't be established.
ECDSA key fingerprint is SHA256:jzTVJS8Wup5kp3lXaquoozJ08oeBgwly4xP3grLCL8c.
ECDSA key fingerprint is MD5:30:84:91:d7:08:f2:8e:e8:f2:0b:f1:db:5c:93:1f:dc.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '10.0.1.212' (ECDSA) to the list of known hosts.
Permission denied (publickey,gssapi-keyex,gssapi-with-mic).
[ec2-user@ip-10-0-0-57 ~]$
```
 
### With forwarding (SUCCESS)
```
mylaptop:~ myuser$ ssh -A -i ~/.ssh/3tierTest.cer 35.170.78.188 -l ec2-user

Last login: Tue Dec 27 22:03:07 2022 from redacted.example.com

       __|  __|_  )
       _|  (     /   Amazon Linux 2 AMI
      ___|\___|___|

https://aws.amazon.com/amazon-linux-2/
30 package(s) needed for security, out of 49 available
Run "sudo yum update" to apply all updates.

[ec2-user@ip-10-0-0-57 ~]$ ssh 10.0.1.212

       __|  __|_  )
       _|  (     /   Amazon Linux 2 AMI
      ___|\___|___|

https://aws.amazon.com/amazon-linux-2/
30 package(s) needed for security, out of 49 available
Run "sudo yum update" to apply all updates.

[ec2-user@ip-10-0-1-212 ~]$
```

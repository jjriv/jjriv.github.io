Title: A Simple Guide to Configuring and Hardening Ubuntu Server
Slug: ubuntu-simple-configuration-hardening
Date: 2024-06-04
Category: articles
Tags: Linux, Ubuntu, Security
Summary: This tutorial covers the bare minimum configuration and hardening you should apply to any new Ubuntu server.
<!-- Status: draft -->

When you deploy a new Ubuntu server the default installation is not sufficient enough to protect it from getting compromised. This is especially true if you are using a popular cloud provider such as [Linode](https://linode.com) or [Digital Ocean](https://digitalocean.com). Hackers are known to monitor the IP addresses they own, like sharks circling a reef looking for easy prey. Your newly created server will start getting scanned for vulnerabilities and sprayed with login attempts the moment it comes online. 

This tutorial teaches you how to configure and harden Ubuntu server in a few simple steps. The purpose of this guide is to create a baselined server from which you can build your project. 

###Step 1: Create Your User Account

The first thing to do is to create a non root user you can use to log in to the server. Run the following command and answer each of the prompts. 

```
adduser jjriv
```
Give your account to have sudo access so that it can run commands as root. This is necessary so that you can configure the server and install software without directly accessing the root account.
```
usermod -aG sudo jreeve
```

###Step 2: Create and Install your SSH Key

The problem with password based authentication is that it&rsquo;s too easy for hackers to guess your password. It&rsquo;s better to use a SSH key, which is a lot like a key you would use to unlock a door. Unless you have the right key, the server won't let you in. 

To create the key pair, run the following command on the server from which you&rsquo;ll be connecting. Follow the prompts, giving your key a unique name to distinguish it from other keys you might have already created. When prompted for a passphrase, leave it blank.
```
ssh-keygen
```

The results of running the command should look like this:
```
$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/jjriv/.ssh/id_rsa): /home/jjriv/.ssh/ubuntu
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/jjriv/.ssh/ubuntu
Your public key has been saved in /home/jjriv/.ssh/ubuntu.pub
The key fingerprint is:
SHA256:ApsRGMJE28I9Rp3WfBvfgIokmNvKD5hRP5FYa4Nevq4 jjriv@lucinda
...
```

Now that you&rsquo;ve created your private and public key pair, the public key needs to be uploaded to the server. Run the following command and enter your password when prompted. Be sure to use the username and IP for your server.

```
ssh-copy-id -i /home/jjriv/.ssh/ubuntu jjriv@1.2.3.4
```

To verify your key is working, log in with it using the command below. If it was successful, you will be logged in without being asked to enter a password.
```
ssh -i /home/jjriv/.ssh/ubuntu jjriv@1.2.3.4
```

###Step 3: Configure SSH to Prevent Password and Root Login

As mentioned above, password based logins are insecure. Therefore, we want to configure the server to not allow anyone to log in with anything other than a key. We also want to make it so that the root user can not log in. Here&rsquo;s how.

Create a sshd config file:
```
vim /etc/ssh/sshd_config.d/lockdown.conf
```

Add the following two directives. THe first one tells the server to not let the root user log in. The second one tells the sever not to allow any password based logins.
```
PermitRootLogin no
PasswordAuthentication no
```

Then restart sshd:
```
systemctl restart sshd
```

###Step 4: Configure the Server&rsquo;s Firewall with UFW

UFW stands for [Uncomplicated Firewall](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-ubuntu). It&rsquo;s the default firewall configuration tool for Ubuntu and was created to make security more user friendly. It works by walling off your server from all public access, except for the doors you allow it to keep open. 

UFW is turned off by default. We want to configure it so that it&rsquo;s automatically enabled and blocking every point of entry except one, SSH.

To accomplish that, we&rsquo;ll add a rule that allows SSH:
```
sudo ufw allow ssh
```

Now that we&rsquo;ve created a firewall rule for SSH, let&rsquo;s enable UFW:
```
ufw enable
```

Now that you&rsquo;ve disallowed root login, disallowed password authentication, and configured UFW, your server can only be accessed by someone with your username and key. This reduces the number of attack vectors exponentially.

###Step 5: Update the Server

Keeping your server updated is equally as important for keeping it safe. Ubuntu will automatically apply any security patches by default. However, it&rsquo;s best practice to regularly update all of the packages installed on your server.

To do this, we just need to run the package manager update with the following two commands:
```
sudo apt update
sudo apt upgrade
```

###Step 6: Set the Hostname and Timezone

Now that the server is secured we can tailor it by setting the hostname and timezone. To set the hostname, run this command:
```
hostnamectl set-hostname myhost.jjriv.com
```
Then change the timezone to your location:
```
timedatectl set-timezone America/Los_Angeles
```

###Keep Security in Mind Going Forward

Now that your server is configured and secure, it&rsquo;s important to maintain a security mindset going forward. As you add more services, such as a web server, you&rsquo;ll be opening up more than just a port. The service running on that port will also be exposed, including any vulnerabilities and misconfigurations it may have. Be sure to research and implement best practices for each and every service you install to keep your server secure.

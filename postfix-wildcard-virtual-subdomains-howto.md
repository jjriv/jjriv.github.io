Title: How to Configure Wildcard Subdomains and Virtual Mailboxes in Postfix
Slug: postfix-wildcard-subdomains-virtual-mailboxes
Date: 2024-05-28
Category: articles
Tags: Linux, postfix
Summary: A tutorial on how to configure Postfix mail server to receive messages addressed to any recipient and subdomain.
<!-- Status: draft -->

This tutorial teaches you how to configure a [Postfix](https://www.postfix.org/) mail server to receive messages addressed to any recipient and subdomain at your domain. 

A common use case for this configuration is a web-based software that receives emails intended for its users. For example, the mail server would accept delivery for messages sent to hello@jane.doe.com and hi@john.doe.com. The messages are then stored by Postfix in a single mailbox for further processing. 

Let's get started.

###Step 1: Configure DNS

DNS takes a while to propagate, so set this up first. We need to create A and MX records for your domain. You will need to know the IP address of your mail server. It should look like this:

```
Record type: A
Name: mx.example.com
TTL: 300
IP: 1.2.3.4
```
```
Record type: MX
Name: *.example.com
TTL: 300
Priority: 10
Server: mx.example.com
```

To test if the DNS updates for the these records have propagated to your servers, run the following commands:
```
dig A mx.example.com
...
;; QUESTION SECTION:
;mx.example.com.     IN      A

;; ANSWER SECTION:
mx.example.com. 0    IN      A       1.2.3.4
```

```
dig MX subdomain.example.com
...
;; QUESTION SECTION:
;subdomain.example.com.     IN      MX

;; ANSWER SECTION:
subdomain.example.com. 0    IN      MX      10 mx.example.com.
```


###Step 2: Create a user and group for sharing read and write access to the mailbox

Postfix will store incoming emails in a single text-based mailbox file on the server. We need to create a user and group and assign them ownership of the file, so that both our Postfix server and web application can access the same mailbox. 

First, create a group called vmail:
```
groupadd -g 5000 vmail
```

Second, we create a user belonging to the vmail group, with no login access and the mail directory as its home.
```
useradd -g vmail vmail
usermod -d /var/mail vmail
usermod vmail -s /sbin/nologin
usermod -aG mail vmail
```

Third, we create the mailbox file, give ownership of it to our newly created vmail user, and set the permissions:
```
touch /var/mail/vmail
chown vmail:vmail /var/mail/vmail
chmod 0660 /var/mail/vmail
```

When you are finished, you should see something like this in the `/var/mail` directory:
```
-rw-rw----  1 vmail vmail  556 Mar 13 13:03 vmail
```

###Step 3: Grant mailbox access to the cron user (optional)


If your web application will be acessing the mailbox through a scheduled cron, you need to add the cron user to the vmail group, which will grant it read/write access. 
```
usermod -aG vmail cronuser
```

Additionally, Linux distributions, such as Ubuntu, will block the cron user from accessing a virtual mailbox file outside of its home directory due to security restrictions. To resolve this we add a symbolic link in the cron user&rsquo;s home directory:
```
ln -s /var/mail/vmail /home/cronuser/vmail
```

###Step 4: Configure Postfix to handle virtual subdomains

The final step is to configure Postfix to accept email being sent to wildcard virtual subdomains. To accomplish this we use the virtual mailbox features of postfix and a few regular expressions to enable wildcards for our subdomains.

First, use your favorite text editor and create the virtual mailbox domains file `/etc/postfix/vdomains`. This file contains the regular expression for matching the recipient&rsquo;s email subdomain to a valid virtual domain.
```
/((\w[\w\-]*)\.)+example\.com/ OK
```

Second, create the virtual mailbox maps file `/etc/postfix/vmailbox`. This file contains the regular expression needed to map the wildcard domain to our virtual mailbox.
```
/@((\w[\w\-]*)\.)+example\.com/ vmail
```

Third, run the following commands to update the Postfix configuration. These changes instruct Postfix to use the domains and maps files for parsing and delivering mail to our virtual mailbox file. They also establish ownership of our virtual mailbox file. 

```
postconf -e 'virtual_mailbox_domains = pcre:/etc/postfix/vdomains'
postconf -e 'virtual_mailbox_base = /var/mail'
postconf -e 'virtual_mailbox_maps = pcre:/etc/postfix/vmailbox'
postconf -e 'virtual_minimum_uid = 5000'
postconf -e 'virtual_uid_maps = static:5000'
postconf -e 'virtual_gid_maps = static:5000'
postconf -e 'virtual_mailbox_limit = 0'
postconf -e 'virtual_mailbox_lock = fcntl'
```

*Note: The virtual_mailbox_limit is set to zero here to denote that the mailbox filesize should not be restricted. You can set this value to whatever you would like.*

Finally, create a lookup table file for the virtual domains and maps and then reload postfix:
```
postmap /etc/postfix/vdomains
postmap /etc/postfix/vmailbox
postfix reload
```

That&rsquo;s it! Emails should start arriving on the server, assuming DNS has propagated. It&rsquo;s up to you what you do next. For web applications that utilize a scheduled cron look into the IMAP libraries available for the scripting language you are using.

###Resources

I could not have figured any of this out without others having already done most of the work. Here is a list of articles, forum discussions, and general documentation that helped me out.

* [Postfix Virtual Domain Hosting Howto](https://www.postfix.org/VIRTUAL_README.html)
* [pcre_table – format of Postfix PCRE tables](https://www.postfix.org/pcre_table.5.html)
* [DNS and sendmail: 21.3.4 Wildcard MX Records](https://www.c3.hu/docs/oreilly/tcpip/sendmail/ch21_03.htm#SML2-CH-21-SECT-3-4)
* [Wildcard Records](https://en.wikipedia.org/wiki/Wildcard_DNS_record)


###Notes
1. If the mail server is behind a firewall, you will need to open up port 25 so that postfix can receive mail. 
2. This post assumes you already have Postfix installed, secured, and sending emails.
4. We are using PCRE for the regular expression matching, which requires a replacment string. Since the vdomains file is only looking for a match, we can use any arbitrary replacement string. In our example, we simply use &ldquo;OK.&rdquo; An empty string works as well. If the replacement string is not specified, the pattern matching will still work, however, warnings will show up in the mail logs.
6. If you&rsquo;d like to filter and reject certain incoming emails, the [header_checks](https://www.postfix.org/header_checks.5.html) configuration parameter is a great place to start.
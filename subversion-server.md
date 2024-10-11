Title: How to Configure and Deploy Subversion Server on Ubuntu
Slug: subversion-server
Date: 2024-05-05
Category: articles
Tags: Linux, Ubuntu, Version Control, Subversion
Summary: This tutorial shows you how to configure Apache to run as a Subversion server on the Ubuntu Linux operating system.
<!-- Status: draft -->

Subversion is an open source version control system developed by Apache since the year 2000. Even though it&rsquo;s been largely overshadowed by Git, the [Subversion software project](https://subversion.apache.org/) is still [widely used](https://get.assembla.com/blog/apache-subversion-still-used/). Subversion is an ideal choice for companies that have strict compliance and security standards. In other words, companies that want full control of their version control.   

This tutorial teaches you how to install and configure Subversion server on the Ubuntu operating system.

###Step 1: Install the Apache and Subversion packages

Run the following commands to install Apache and Subversion.

```
apt install apache2
apt install subversion
```

###Step 2: Enable Apache DAV modules

The DAV modules are necessary to enable WebDAV, which is a set of extensions to the HTTP protocol which allow users to collaboratively edit and manage files on a web server. Use the apache `a2enmod` command to enable the dav modules

```
a2enmod dav
a2enmod dav_fs
a2enmod dav_svn
```

###Step 3: Configure the Subversion Virtual Host

Create the following file:  
`/etc/apache2/sites-available/subversion.conf`

Add the following virtual host configuration:

```
<IfModule mod_ssl.c>
<VirtualHost *:443>
    ServerName jjriv.com
    DocumentRoot /var/www/html
    KeepAlive On
    MaxKeepAliveRequests  1000
    KeepAliveTimeout 15
    CustomLog /var/log/apache2/svn_logfile "%t %u %{SVN-ACTION}e" env=SVN-ACTION
</VirtualHost>
</IfModule>
<VirtualHost *:80>
    ServerName jjriv.com
    DocumentRoot /var/www/html
</VirtualHost>
```

###Step 4: Secure the Subversion Virtual Host

Since DAV access allows users to manipulate files on the server, you must assure that your server is secure. The virtual host should be protected by authentication. [HTTP Digest Authentication is the preferred method](https://httpd.apache.org/docs/2.4/mod/mod_dav.html#security). However, Basic Authentication is acceptable if the site is accessed over an SSL enabled connection.

Add the following configuration to your config file:  
`/etc/apache2/sites-available/subversion.conf`

```
<Location /svn>
  DAV svn
  SVNParentPath /var/svn

  # Authentication: Digest
  AuthName "Subversion repository"
  AuthType Digest
  AuthDigestProvider file
  AuthUserFile /etc/svn-auth.htdigest

  # Authorization: Authenticated users only
  Require valid-user
</Location>
```

The above configuration will read in a file containing usernames and encrypted passwords. That file can be created using the following command. The first time you run it, you need to use the `-c` option to instruct htdigest to create a new file:
```
$ htdigest -c /etc/svn-auth.htdigest "Subversion repository" jjriv
Adding password for jjriv in realm Subversion repository.
New password: *****
Re-type new password: *****
```
To add more users, run htdigest without the create option:
```
$ htdigest /etc/svn-auth.htdigest "Subversion repository" john
Adding user john in realm Subversion repository
New password: *******
Re-type new password: *******
```

###Step 5: Configure the Firewall

Assuming you've secured your server already, you will need to open up ports for http and https. Run the following command to update the firewall using the UFW utility.

```
ufw allow http
ufw allow http
```

###Step 6: Reload Apache

The next step is to load all of our configuration changes by reloading Apache:

```
systemctl reload apache2
```

###Step 7: Install SSL Certificate from Let&rsquo;s Encrypt

For the Subversion server to be secure it needs to have a SSL certificate installed and redirect all http traffic to https. Fortunately, [certbot](https://certbot.eff.org) from [Let&rsquo;s Encrypt](https://letsencrypt.org/) makes this really easy.

First, [install certbot using the Apache / Ubuntu instructions](https://certbot.eff.org/instructions?ws=apache&os=ubuntufocal).

Next, run the following command and answer the prompts to create your certificate:

```
certbot --apache -d jjriv.com
```

This will install the certificate, configure the redirect, and restart Apache. You can use curl to verify that it completed successfully.


To verify the http to https redirect:
```
$ curl -I http://jjriv.com
HTTP/1.1 301 Moved Permanently
Date: Thu, 06 Jun 2024 02:51:52 GMT
Server: Apache
Location: https://jjriv.com/
...
```

To verify the certificate (this command will fail if the cert is not installed correctly):
```
$ curl -I https://jjriv.com
HTTP/1.1 401 Unauthorized
Date: Thu, 06 Jun 2024 02:52:38 GMT
Server: Apache
...
```

Now, point your browser at your domain and log in using the credentials you created. If you&rsquo;re not sure what to do next, start by reading the [Getting Data into Your Repository](https://svnbook.red-bean.com/en/1.7/svn.tour.importing.html) chapter of the Subversion book.






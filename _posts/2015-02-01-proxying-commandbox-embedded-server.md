---
layout: post
status: publish
published: true
title: Reverse Proxy Your CommandBox Embedded Server - Apache
description: "Developing and Deploying Coldbox Applications using the CommandBox Embedded Server - Part 1"
author:
  display_name: Jon Clausen
author_email: jon_clausen@silowebworks.com
author_url: http://jonclausen.com
categories:
- Programming
tags:
- Coldbox
- Coldfusion
- Lucee
- CFML
- CommandBox

fa_class: "fa-gears"
---
I'm a big fan of the [CommandBox][1] embedded [Lucee server][2] and am finding the scenarios in which I use it for development and testing to be growing day-by-day. 

At some point, though, you need to be able to develop applications for production in an environment that mimics the production environment, including full usage of rewrite rules, routing based on domain names, etc.  

I also don't like exposing my app server technology in my URL's so having a path like "/index.cfm/handler/action" upsets my delicate sensibilities.  

So, with that in mind, let's place our Coldbox embedded server behind Apache.  First things first, we'll probably need to modify our hosts file to recognize our local domain.  I'm on a Mac, so my hosts file is located at /etc/hosts, but you can also [make this modification on Windows](http://www.thewindowsclub.com/hosts-file-in-windows).  In your hosts file, add the following:

<pre>
127.0.0.1 mydomain.localhost
</pre>

Next you'll want to start up your commandbox server:

<pre>
$ cd /Sites/mydomain
$ box server start
</pre>

Your browser will open, but you can close that out for now and you'll see something like the following in the terminal output:

<pre>
The server for /Sites/mydomain is starting on 127.0.0.1:63616... type 'server status' to see result
</pre>

We're going to need that port number so that we can fix it permanently so, stop your server and add that port to your box.json as the default port:

<pre>
$ box package set defaultPort=63616
Set defaultPort = 63616
</pre>

Now that we have the default port set permanently, we're going to set up Apache to proxy it. We're going to need mod_proxy enabled on Apache so make sure the following two lines (with your paths) are uncommented in httpd.conf:

<pre>
LoadModule proxy_module libexec/apache2/mod_proxy.so
LoadModule proxy_connect_module libexec/apache2/mod_proxy_connect.so
</pre>

Depending on your Apache installation and your OS, your virtual host files will be in different locations.  I keep all mine in /etc/apache2/vhosts.d with an `include /etc/apache2/vhosts.d/*.conf` in my httpd.conf file, which loads all config files in that directory on restart. Let's create our domain config file:

<pre>
$ sudo vi /etc/apache2/vhosts.d/mydomain.localhost.conf 
</pre>

This will open up a new file for editing.  We're using sudo so that we can save it, because OSX's permissions normally wouldn't let us do that in the /etc/ path.  Now add the following (we're copying a big segment of the default coldbox .htaccess file, with one exception (see the comment in the config code):

{%highlight diff%}
# My Application
<VirtualHost 127.0.0.1:80>
ServerName mydomain.localhost
# Document Root
DocumentRoot /Sites/mydomain/
<Location / >
	RewriteEngine on
	RewriteBase /

# Bypass images, css, javascript and docs, add your own extensions if needed.

	RewriteCond %{REQUEST_URI} \.(bmp|gif|jpe?g|png|css|js|txt|pdf|doc|xls|ico)$
	RewriteRule ^(.*)$ - [NC,L]

# The ColdBox index.cfm/{path_info} rules.

	RewriteRule ^$ index.cfm [QSA,NS]
	RewriteCond %{REQUEST_FILENAME} !-f
	RewriteCond %{REQUEST_FILENAME} !-d

# Notice the [P] flag at the end of the rewrite rule. 
#	This tells Apache that the rewrite rule is proxy

	RewriteRule ^(.*)$ http://127.0.0.1:63616/index.cfm/%{REQUEST_URI} [P]	

# For development, you can just keep the following 
# in place to provide you with full access.
# What's below tells apache to proxy everything 
# that hasn't been found with the rules above. 
 
	ProxyPass http://127.0.0.1:63616/
	ProxyPassReverse http://127.0.0.1:63616/
	SetEnv force-proxy-request-1.0 1

# Now for our permissions
# Note: "Require all granted" is Apache 2.2 syntax
# for <2.2, you'll want Order allow,deny in its place

	AllowOverride All
	Require all granted
	Allow from all

</Location>
</VirtualHost>
{%endhighlight%}
Now restart Apache:
<pre>
sudo httpd -k restart
</pre>

and point your browser to: http://mydomain.localhost Paths like http://mydomain.localhost/myhandler/myaction will now work as they should. You can also configure additional require rules and proxies (like NodeJS) in the file above and mimic your production environment.  Vóila! Code like the wind. -JC

[1]:http://www.ortussolutions.com/products/commandbox "CommandBox"
[2]:http://lucee.org/

---
layout: post
status: publish
published: true
title: Logrotate Issues after Linux Distribution Upgrade
description: "The one in which your web and database server log files stop rotating properly"
author:
  display_name: Jon Clausen
author_email: jon_clausen@silowebworks.com
author_url: http://jonclausen.com
categories:
- Linux
tags:
- SuSE
- Linux
- Logging

fa_class: "fa-exclamation-circle"
---

I received a dreaded "The web site is down!" call from a client this morning.  I had stepped away from the computer and when I sat down to take a look at the problem my inbox was full with site outage notifications.  

The VPS was non-responsive so I had to force reboot the server from the VMWare master.  None of the other VM's on the server were affected - just this one, which is running a <abbr title="Linux,Apache,(NGINX),PHP,PostgreSQL">LA(N)PP</abbr> stack on [SuSE 13.3](https://www.opensuse.org/en/).  Once I got the server back up and running again, it was time to dig in to the logs to see what had caused the issue. 

A look at /var/log/messages showed the last item logged before the system became non-responsive was an entry for [syslog-ng](http://en.wikipedia.org/wiki/Syslog-ng), and then nothing until the reboot. Next I went and looked at the web server logs.  

This particular machine uses NGINX as a reverse proxy for Apache. NGINX handles the static files (which it does really, really well) on Port 80 and high-level request caching, while Apache handles serving PHP and all requests on Port 443 (There is a method to this madness, which I'll discuss in a future post).  In addition, NGINX also reverse proxies to three other servers so the log for that site logs every requests to those subapps running on other servers. 

The NGINX log for the site showed a filesize of nearly 1GB, which shouldn't have happened, as rotation happens daily, plus the max filesize on that log is set at 4MB. Big problem.

Running logrotate in debug mode manually, `logrotate -dv /etc/logrotate.conf`, returned the following message regarding the NGINX log (along with similar messages for few others):

{% highlight diff %}
error: skipping "/var/log/nginx/mysite.com.access.log" because parent directory has insecure permissions (It's world writable or writable by group which is not "root") Set "su" directive in config file to tell logrotate which user/group should be used for rotation.
{% endhighlight %}

Root cause found.  After some digging in to the whys, it turns out the version of logrotate on SuSE 13.3, which this machine was upgraded to two weeks ago now requires explicit directory permissions to be specified in the logrotate command unless the the directory is chowned root:root.  [This change was introduced in logrotate version 3.8.0](https://fedorahosted.org/logrotate/browser/tags/r3-8-0/CHANGES) so the this issue applies to all linux distros using that version or above. 

On this system NGINX runs in group "nginx" and Apache runs in group "www" so adding the following to the configuration block in /etc/logrotate.d/nginx fixed the problem: `su root nginx`.

The final configuration block looks like:

{% highlight diff %}
/var/log/nginx/*.log {
	su root nginx
	compress
	dateext
	maxage 365
	rotate 99
	size=+4096k
	notifempty
	missingok
	create 644 root root
	postrotate
	  /etc/init.d/nginx reopen
	endscript  
}
{% endhighlight %}

I also had to change all of the Apache logrotate entry blocks to `su root www` as the ownership permissions on those directories need to be writeable by the group in which httpd runs.  If you're running MySQL/MariaDB, you'll need to change add `su root mysql` to any config blocks in /etc/logrotate.d/mysql as well.  PostgreSQL handles its own log rotation, so those will not be an issue.

Since this applies to all Linux distros running logrotate 3.8.0 and above, it's worthy of a quick check of your systems, if you've upgraded recently.  Running `ls -l` from within your /var/log directory should allow you to pull the correct group names to ensure log rotation will keep functioning properly.  If you started with a fresh machine running those versions, your package manager installation will add those blocks automatically.

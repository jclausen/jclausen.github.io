---
layout: post
comments: true
status: draft
published: false
title: Deploying on Windows Server With Capistrano
description: "The one in which we setup automated deployment, with Capistrano, to Windows Server"
author:
  display_name: Jon Clausen
author_email: jon_clausen@silowebworks.com
author_url: http://jonclausen.com
categories:
- Web Development
tags:
- Capistrano
- Windows
- deployment
- coldfusion
- coldbox
---

I've recently been moving more and more apps to [Capistrano][] deployment.  I shopped around for different options for awhile before I settled on Cap.  [Mina][] was an initial front-runner and there are those who greatly prefer it because of Capistrano's built-in performance bottlenecks, these are part of its failsafe and redundancy strategies.  Rather than viewing those as a turn-off, I chose it *because* of those strategies.

[Capistrano]: http://capistranorb.com/
[Mina]: http://nadarei.co/mina/

Windows
-------------------
While setting up your apps for deployment on Linux is straightforward and a [metric ton of documentation](http://guides.beanstalkapp.com/deployments/deploy-with-capistrano.html) exists to help with that, deploying on Windows isn't so straightforward.  Unix-based deployment relies on symbolic links handle deal with serving the current revision, as well as shared directories and files.  Those symlinks aren't compatibile with Windows, so we have to deal with that behavior.  

In addition, Windows doesn't do SSH out of the box, which is how the Capistrano remote actions are performed, so we have to deal with enabling that.  Enter [Cygwin](https://cygwin.com/), which we'll discuss more in a moment. 

Step 1 - Install Cygwin
-----------------------
We need SSH access before we can do anything so first we install Cygwin.  [Download the installer](https://cygwin.com/install.html) for your architecture and install the following packages, in addition to the default installation:

* openssh
* Git or Subversion, depending on your SCM
* rsync (installed by default at the present, but double-check during install)

I would aslo recommend the following packages, which aren't included in the default  installation (or roll your own install - [full list of packages here](https://cygwin.com/packages/package_list.html)):

* wget (handy for file downloads)
* curl (more options than wget if you need to create custom web hooks)
* ruby 
* rubygems

*Note: if you're familiar with using a Linux package manager, you may want to check out [cyg-apt](https://code.google.com/p/cyg-apt/), which is an apt-get style package manager for Cygwin*

Once your installation is complete, open a Cygwin terminal, and run 

{% highlight diff %}ssh-host-config{% endhighlight %}

You'll be prompted with a series of questions.  Answer them as so:

{% highlight diff %}
*** Query: Should privilege separation be used? <yes/no>: yes
*** Query: New local account 'sshd'? <yes/no>: yes
*** Query: Do you want to install sshd as a service?
*** Query: <Say "no" if it is already installed as a service> <yes/no>: yes
*** Query: Enter the value of CYGWIN for the deamon: [] binmode ntsec
*** Query: Do you want to use a different name? (yes/no) yes/no
{% endhighlight %}

If you want to use the same name, that is cyg_server, enter no and answer the following:

{% highlight diff %}
*** Query: Create new privileged user account 'cyg_server'? (yes/no) yes
*** Query: Please enter the password:
*** Query: Renter:
{% endhighlight %}

If you want to use a different name, enter yes and answer:

{% highlight diff %}
*** Query: Enter the new user name: cyg_server1
*** Query: Reenter: cyg_server1
*** Query: Create new privileged user account 'cyg_server1'? (yes/no) yes
*** Query: Please enter the password:
*** Query: Reenter:
{% endhighlight %}

If the configuration is successful, you will see the following message:

{% highlight diff %}Host configuration finished. Have fun!{% endhighlight %}
	

Create any user accounts (and ensure those users have directory permissions for your deployment directories).  I chose to create a user account called 'deploy':

{% highlight diff %}useradd deploy{% endhighlight %}


Step 2 - Setup your Project for Capistrano
------------------------------------------



{% highlight ruby %}
# Disable default symlink behavior
before "deploy:started", "windows_server:no_symlinks"
# Sync shared directories
after "deploy:started", "windows_server:upsync_shared"
after "deploy:publishing","windows_server:downsync_shared"
after "windows_server:downsync_shared", "windows_server:sync_release"
after "windows_server:sync_release","coldfusion:windows:restart_jetty"
# Re-Initialize our Coldbox Application
after "coldfusion:windows:restart_jetty","coldbox:reinit"
{% endhighlight %}


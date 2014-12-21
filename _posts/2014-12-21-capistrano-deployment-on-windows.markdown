---
layout: post
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

I've recently been more and more apps to deployment with [Capistrano][].  I shopped around for different options for awhile before I settled on Cap.  [Mina][] was an initial front-runner and while there are [those who say Capistrano is dead](https://weluse.de/blog/capistrano-is-dead-use-mina.html) because of built-in performance bottlenecks, that are part of its failsafe and redundancy architecture.  


[Capistrano]: http://capistranorb.com/
[Mina]: http://nadarei.co/mina/

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


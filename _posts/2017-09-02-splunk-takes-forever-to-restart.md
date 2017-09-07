---
layout: post
title: "Fixing Splunk When it Takes Forever to Restart"
date: 2017-09-02 14:53:00 CST
author: jd
categories: [splunk restart troubleshooting configuration]
published: true
comments: true
---
![splunkd restart taking forever](/images/Screen%20Shot%202017-09-02%20at%202.21.46%20PM.png)
![splunkweb taking forever to start](/images/Screen%20Shot%202017-09-02%20at%202.22.00%20PM.png)

Are you running a version of Splunk Enterprise prior to 6.6.3 and the restarts are taking forever? I feel your pain. If I wasn't at work, I'd probably take a nap while waiting for the services to come back up.

<!--more-->

The good news is that fixing the very long restart times is really simple. If allowing production servers to touch the internet is something you want or can do, then adding a quick proxy `export` statement to the profile of the user running Splunk. This will allow Splunk to talk to the update URLs.

Proxy or not, if the thought of allowing Splunk to reach the internet gives you the willies, here's what you should do. Add `QUICKDRAW_URL = 0` to the bottom of `splunk-launch.conf`. You'll find that config file in `/opt/splunk/etc` or `$SPLUNK_HOME/etc` if Splunk was installed using the default location.

Restart Splunk but don't blink. You might miss it.

### Before:
![](/images/Screen%20Shot%202017-09-02%20at%202.21.46%20PM.png)

![](/images/Screen%20Shot%202017-09-02%20at%202.22.00%20PM.png)

### After:

![](/images/Screen%20Shot%202017-09-02%20at%202.22.16%20PM.png)

![](/images/Screen%20Shot%202017-09-02%20at%202.23.09%20PM.png)

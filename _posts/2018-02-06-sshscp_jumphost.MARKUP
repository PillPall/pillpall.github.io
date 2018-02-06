---
layout: post
title:  "Managing ssh connection with an jumphost in between"
date:   2018-02-06 12:00:00 +1000
categories: Other
---

My useful SSH/SCP commands to work with a Jumphost, but never want to touch it.

Open a Shell on the remot host
To connect a Unix machine which is only accessible via a jumphost directlty, you need to store your public key in the `~/.ssh/authorized_key` file on the jumphost.  Afterwards you can access the remote host using following commands:

{% highlight Bash %}
# example command
/bin/bash># ssh -J jumpuser@JUMPHOST remoteuser@REMOTEHOST

# Command to connect directly to Host 10.0.0.1 which 
# is only accessible via Jumphost 93.184.216.34
/bin/bash># ssh -J cadmin@93.184.216.34 zadmin@10.0.0.1
[zadmin@ip-10-0-0-1 ~]$ 
{% endhighlight %}

Port Forwarding
To access a Port on the remote Host we can use the buildin feature of TCPForwarding in OpenSSH. This allows us for example to open a HTTP website on our localhost which runs on the remote server.

At first we need to enable TCPForwarding in the Daemon configurationfile of SSH `sshd_config`, per default TCPForwarding is enabled at least since OpenSSH Version 7.6p1. 

* Open Port 80 
{% highlight Bash %}
# example command
/bin/bash># ssh -L LOCAL_PORT:WEBSERVER:REMOTE_PORT -J jumpuser@JUMPHOST remoteuser@REMOTEHOST

# Command to connect directly to Host 10.0.0.1 and 
# open port 80 on localhost and connect it to port 80
# on the remote HOst
/bin/bash># ssh -L 80:10.0.0.1:80 -J cadmin@93.184.216.34 zadmin@10.0.0.1
[zadmin@ip-10-0-0-1 ~]$
{% endhighlight %}
As long as you keep the session open, you are now able to open up your favourite browser and type in following as URL: `http://localhost:80`
It should open the Website of the connecting remote host. Keep in mind you can do it with every Port(service) which runs on the remote host and listen to a network Port.

If you get an error like this:

{% highlight bash %}
channel 3: open failed: connect failed: Connection refused
channel 3: open failed: connect failed: Connection refused
channel 3: open failed: connect failed: Connection refused
{% endhighlight %}
Check if the service is really running on the remote host or if the port is listening `netstat -tlnp`.

Filetransfer with Jumphost
We can use `scp` to transfer files/ directories to our remotehost via the jumphost, similar to the first example to access the remotehost directly.

{% highlight bash %}
# example command files
/bin/bash># scp -oProxyJump=jumpuser@JUMPHOST SOURCE_FILE remoteuser@REMOTEHOST:DEST_FILE
# example command directories
/bin/bash># scp -r -oProxyJump=jumpuser@JUMPHOST SOURCE_PATH remoteuser@REMOTEHOST:DEST_PATH

/bin/bash># scp -oProxyJump=cadmin@93.184.216.34 ./index.html zadmin@10.0.0.1:/var/www/html/
/bin/bash># scp -r -oProxyJump=cadmin@93.184.216.34 ./gifs zadmin@10.0.0.1:/var/www/html/gifs
{% endhighlight %}

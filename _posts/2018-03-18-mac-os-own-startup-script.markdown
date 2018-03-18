---
layout: post
title:  "Create own startup script for MacOSX Sierra"
date:   2018-03-18 10:00:00 +1000
categories: MacOS
---

A small description how to create your own Bash script which got's loaded after login.
<!--excerpts-->

I want to automate the start of a Virtual Machine in VirtualBox after I login and additionally to mount a NFS share which got's exported within my Virtual Machine. Also the opposite when I shutdown/restart my Laptop.

I use my Virtual Machine for several reasons but mainly to develop Software as I don't like to install all necessary tools and dependencies on my Laptop OS. So, to run all make commands I connect to my VM via SSH and use the NFS Export to manage and edit all files and repos on my Laptop OS.

Since MacOSX Sierra use launchd for starting/stopping daemons, applications, processes etc. we need to create a launchd Property List File. More information about creating Launchd jobs in MacOSX can be taken from the the Apple Developer Documentation [here(link)](https://developer.apple.com/library/content/documentation/MacOSX/Conceptual/BPSystemStartup/Chapters/CreatingLaunchdJobs.html).

## launchd Property List File

The Property List file contains all information about our script we want to run at startup/shutdown in XML Syntax. As taken from the documentation, following property list keys are required:

Key                  | Description
-------------------- | --------------------
**Label**            | *Contains a unique string to identify your daemon to launchd.*
**ProgramArguments** | *The arguments used to launch your daemon.*
                     |

For a complete list of all available keys have a look into the man page `man launchd.plist`.

My launchd Property List File `com.user.virtualbox.startvm.plist`:
{% highlight xml linenos %}
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
  <dict>
    <key>Label</key>
    <string>com.user.virtualbox.startvm</string>
    <key>ProgramArguments</key>
    <array>
	    <string>/usr/local/bin/start_vm.sh</string>
    </array>
    <key>RunAtLoad</key>
    <true/>
    <key>LaunchOnlyOnce</keyu>
    <true/>
    <key>StandardOutPath</key>
    <string>/Users/mbloch/log/startvm_stdout.log</string>
    <key>StandardErrorPath</key>
    <string>/Users/mbloch/log/startvm_errout.log</string>
  </dict>
</plist>
{% endhighlight %}

Now that Property List file exist we need to consider when we want to run the script.

When                    | From where (Path)             | Type           
------------------------|-------------------------------|---------------
Sytem Startup/ Shutdown | /System/Library/LaunchDaemons | System-wide daemons provided by Mac OS X.
                        | /Library/LaunchDaemons        | System-wide daemons provided by the administrator.
Login / Logout          | /System/Library/LaunchAgents  | Per-user agents provided by Mac OS X.
                        | /Library/LaunchAgents         | Per-user agents provided by the administrator.
                        | ~/Library/LaunchAgents        | Per-user agents provided by the user
                        |                               |

Since I want to start my script after login and before logout I placed my Property List file under `~/Library/LaunchAgents/com.user.virtualbox.startvm.plist`.

Now that we have our Property List file we need to add it to launchd. To avoid errors while adding it to launchd like. If you place it under the User folders `~/Library/LaunchAgents` you have to make sure the permissions are set to the user. For the other folders make sure you use `root:wheel`.

Run
{% highlight Bash %}
/bin/bash># launchctl load -w ~/Library/LaunchAgents/com.user.virtualbox.startvm.plist
{% endhighlight %}
The `-w` option will enable our service directly.

## Startup/Shutdown script
Now that we told our OS about our new Startup/Shutdown script we need to create it.

Basically you can use a simple Hello world script as a startup script. But if you want to make sure it runs also during shutdown/logout you need to run this script continuously, till shutdown/logout.

I realised my script to start/stop the VMs and to mount/umount the NFS share with 6 functions:
* Start VM
* Stop VM
* Mount NFS
* UMOUNT NFS
* Handler
* Run in Background

{% highlight xml linenos %}
#! /bin/bash
#
# Startup script for MacOSX Sierra to start and stop a Virtual Machine
# everytime this scripts gots executed or receives a SIGTERM SIGKILL SIGINT SIGHUP Signal
#

# Variables
VM_ID=d9782aed-e997-42b1-96f8-1cdb1ee3d55c
Mount_PATH=/Users/mbloch/projects
NFS_IP=192.168.98.101
NFS_PATH=/data/projects
NFS_OPTS=noowners,nolockd,resvport,hard,intr,rw,tcp,nfc

StartVM()
{
        echo "Start VM"
        /usr/local/bin/VBoxManage startvm ${VM_ID} --type headless
        echo ${?}
	sleep 10
	MountNFS
	echo "All done"
	Background
}

StopVM()
{
	UmountNFS
        echo "shutdown VM"
        /usr/local/bin/VBoxManage controlvm ${VM_ID} acpipowerbutton
        echo ${?}
}

MountNFS()
{
	i=0
	err_nfs=1
	if mount |grep projects > /dev/null; then
		echo "Already mounted"
	else
		while [ ${i} -lt 120 ]; do
			if [ ${err_nfs} -gt 0 ]; then
				echo "Mount NFS"
				sudo /sbin/mount -t nfs -o ${NFS_OPTS} ${NFS_IP}:${NFS_PATH} ${Mount_PATH}
				err_nfs=$(echo $?)
				echo ${err_nfs}
				sleep 1
				i=$[$i+1]
			else
				echo "Mount done"
				i=256
			fi
		done
	fi
}

UmountNFS()
{
        if mount |grep projects > /dev/null; then
      		echo "Unmount NFS"
          sudo /sbin/umount ${Mount_PATH}
        fi
}

Handler()
{
	echo $(date)
	#
	# Condition check if a VM with given VM ID is already running
	# Yes stop it, no start it
	#
	/bin/ps -Af|grep ${VM_ID}|grep -v grep
	err=$(echo $?)
	if [ ${err} -gt 0 ]; then
		StartVM
	else
		StopVM
	fi
}

Background()
{
	tail -f /dev/null &
	wait $!
}

trap StopVM SIGTERM SIGKILL SIGINT SIGHUP

Handler;

{% endhighlight %}

The most important part of this script is the `Background function` and the `trap` line. With `trap` we say call function `StopVM` when a System signals like a shutdown/logout event `SIGTERM` or a kill event `SIGKILL` occurs. But our script can only listen to those events when it's running. That's why we run a endless loop in the background function.

Now after every login my VM get started, the NFS share gets mounted and before logout my NFS Share gets umounted and my VM gets shutdown.

My examples can be found [here(link)](https://github.com/PillPall/examples/tree/master/sierra-launchd-script).

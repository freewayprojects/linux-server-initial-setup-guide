Linux Server Initial Setup Guide
================================

# Overview

This guide is designed to be used after the initial installation of Linux has been completed.  The installation of Linux can be carried out using the [Linux Server Installation Guide](https://github.com/freewayprojects/linux-server-installation-guide).

So, this guide should be used to pick up from where [Linux Server Installation Guide](https://github.com/freewayprojects/linux-server-installation-guide) finished.

For the purposes of this guide we are going to carry out the initial set up on a server which has the FQDN of server1.example.com

We are going to assume that the initial user account which was set up has a username of 'admin'.

# Prerequisites

The initial Linux server installation should have been completed.  This could be any distribution or version of Linux - this guide is intended to be generic as well as having distribution/version specific code snippets.

The main points about the server which should already have been set up are:

* Linux base operating system installed.
* BIOS clock set correctly.
* SSH server installed and available
* Initial administration user created.  This user will be used to login to the server and is separate from the root user.
* Network address configured.
* The hostname and domain should be set up so that the server has a FQDN available.

Basically, the administrator should be able to login to the server over SSH to carry out the intial setup.  This is normally the point at which a hardware server has been put into a rack or located at its permanent location - or when a VM has been built and is available over a network.

# Setup steps

## Logging into the server as root

After the server has had Linux installed it should be possible to log in over SSH (or from a console) under the initial user account - in our examples this user account has the name 'admin'.

Virtually all of the steps required in the initial setup need to be carried out as the root user.  Therefore it is best to switch the ssh session to the root user.

| Distribution | Versions |
| --- | --- |
| Debian | All versions |

    admin@server1:~$ su -

and enter the root user password as set when Debian was installed.

| Distribution | Versions |
| --- | --- |
| Ubuntu | All versions |

    admin@server1:~$ sudo su -

and enter the admin user's password.

## Checking the operating system datetime

In the server installation the server BIOS clock should have been set correctly.  It is very important that the server operating system is reporting the correct datetime and timezone as well.  The package managers, logging tools, certificates etc rely on the server time clock being correct.

### Reading the current datetime

To start with the server datetime should be checked.  On most Linux servers the date and time can be checked by running the date command.

| Distribution | Versions |
| --- | --- |
| Debian | All versions |
| Ubuntu | All versions |
| SLES | All versions |
| RH | All versions |
| Centos | All versions |

~~~
server1admin@server1:~$ date
Thu Jul  4 13:38:36 BST 2013
~~~

### Correcting timezone

If the timezone is not correct then this should be corrected.

| Distribution | Versions |
| --- | --- |
| Debian | All versions |
| Ubuntu | All versions |


    # dpkg-reconfigure tzdata


Then follow the onscreen prompts.

### Changing the server datetime

Obviously, the server BIOS datetime should have been set correctly during the operating system installation.  This should mean that the time reported by the operating system should be roughly correct and take into account any adjustments such as timezone and summer time adjustments.

NB - The server BIOS time should be set to UTC.  In Linux it is the operating system which adjusts datetime for time zones and Daylight Saving Time adjustments.

Later in this set up guide the server will be set up to use internet time servers to keep the server times correct.

If necessary the server datetime can be corrected.

| Distribution | Versions |
| --- | --- |
| Debian | All versions |
| Ubuntu | All versions |


    # date 070413462013


will set the date time to 1:46pm, 4th July, 2013.

##  Make Gnu Screen available

The tool [Screen](http://www.gnu.org/software/screen/) means that commands can be run under a remote session on the server.  This means that if your connection to the server is lost whilst you are carrying out commands then you can log back in to the server and pick up the session again.  This makes it much safer to carry out tasks remotely such as installing software.

### Installing screen

| Disttribution | Versions |
| --- | --- |
| Debian | All versions |


    # apt-get install screen


| Disttribution | Versions |
| --- | --- |
| Ubuntu | 10.04+ |

Screen is installed by default.

### Using Screen

Once screen is installed a session can be created with:


    # screen -DR serveradmin1


If the session is dropped then it can be picked up again with the same command.


    # screen -DR serveradmin1


## Update packages

It's important to make sure that all the server packages are up-to-date.  This is obviously very dependent on the distribution of Linux used - but effectively all Linux distributions use a system of software and package management.

| Disttribution | Versions |
| --- | --- |
| Debian | All versions |
| Ubuntu | All versions |

First of all check that the server is set to use the required repositories.  These are set in the file

    /etc/apt/sources.list

Then the packages list can be updated with:

    root@server1:~# apt-get update

Before updating all the packages it might be an idea to check exactly what will be updated first by looking at the output from:

    root@server1:~# apt-get -s dist-upgrade | less

And then update all packages with:

    root@server1:~# apt-get dist-upgrade

NB - 'dist-upgrade' is preferred to 'upgrade' as it should handle dependencies better.

## Check server reboots

After this initial package update it is a good idea to check that the server reboots OK and that there are no issues with kernels etc.

### Reboot the server

| Disttribution | Versions |
| --- | --- |
| Debian | All versions |
| Ubuntu | All versions |

    root@server1:~# shutdown -r now
    
## Testing the hard disks

The server memory should have been tested when the operating system was installed.  At this point it is an idea to test the hard disks.

There are ways of accessing the hard disk diagnostic information from tools such as Smart Tools and HP's ILO system.  These will give you more direct access to information about hardware status.

What we are going to do here is to use bonnie++ to stress test the HDD's and then monitor the kernel log for errors.  If the disks have been pushed hard for a couple of hours and no errors have been reported in kernel.log then this shows the drives are generally working well.

The advantage of this approach is that it can be used with any hardware or disk configuration.  

It should be noted that hard disks should be monitored with more specific tools if possible.

### Test the HDD's

| Disttribution | Versions |
| --- | --- |
| Debian | All versions |
| Ubuntu | All versions |

Install the tool bonnie++

    root@server1:~# apt-get install bonnie++
	
Then run the benchmarking test with

    root@server1:~# bonnie++ -d /home/admin -u 1000 -x 200
	
Whilst the benchmark test is runnning monitor the output to see if there are any hardware errors with:

    root@server1:~# tail -f /var/log/kern.log
	
	

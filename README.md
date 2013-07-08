Linux Server Initial Setup Guide
================================

# Overview

This guide is designed to be used after the initial installation of Linux has been completed.  The installation of Linux can be carried out using the [Linux Server Installation Guide](https://github.com/freewayprojects/linux-server-installation-guide).

So, this guide should be used to pick up from where [Linux Server Installation Guide](https://github.com/freewayprojects/linux-server-installation-guide) finished.

An important point to note is that this guide is designed to be generic.  Various steps are described - then specific installation steps for various distributions are provided.  It may be possible to script various steps to make server setups quicker and easier - but in general this guide is designed to be more of a check list for points which should be carried out.  

Another point to note is that tools and options have been chosen which are relatively easy to install and set up.

For the purposes of this guide we are going to carry out the initial set up on a server which has the FQDN of server1.example.com.  We are going to assume that the initial user account which was set up has a username of 'admin'.

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

There are various ways of accessing the hard disk diagnostic information from tools such as Smart Tools and HP's ILO system.  These will provide more direct access to information about hardware status.

What we will do is to provide some generic tests which can be applied to most servers.

### Quick test of the hard disk speed

This test will carried out as a quick test to see if the writing speed of the hard disk is reasonable.

| Disttribution | Versions |
| --- | --- |
| Debian | All versions |
| Ubuntu | All versions |

    root@server1:~# dd bs=1M count=512 if=/dev/zero of=test conv=fdatasync

As of 2013 if the write speed is less than approximately 50 MB/s - for example:

    536870912 bytes (537 MB) copied, 20.719 s, 25.9 MB/s

then there may be something which needs to be carried out to increase the speed.

The speed should be more in the region of 200 MB/s.  For example

    536870912 bytes (537 MB) copied, 2.63141 s, 204 MB/s

Once testing has been completed the file 'test' should be removed:

    root@server1:~# rm test

### Detailed test and stress test

What we are going to do first is to use bonnie++ to stress test the HDD's and then monitor the kernel log for errors.  If the disks have been pushed hard for a couple of hours and no errors have been reported in kernel.log then this shows the drives are generally working well.

The advantage of this approach is that it can be used with any hardware or disk configuration.  It should be noted that hard disks should be monitored with more specific tools if possible.

We will then analyse the results from a bonnie++ test and store them in the notes for the server.

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
	
This should be left running for at least a couple of hours.  Any disk errors should show up in the kern.log file.  To stop the test use Ctrl-C.

The CSV output from a bonnie++ run can then be fed into a conversion tool to produce an HTML report which can be put into the server document for reference.

    root@server1:~# bonnie++ -d /home/devback1admin -u 1000
    Using uid:1000, gid:1000.
    Writing a byte at a time...done
    Writing intelligently...done
    Rewriting...done
    Reading a byte at a time...done
    Reading intelligently...done
    start 'em...done...done...done...done...done...
    Create files in sequential order...done.
    Stat files in sequential order...done.
    Delete files in sequential order...done.
    Create files in random order...done.
    Stat files in random order...done.
    Delete files in random order...done.
    Version  1.96       ------Sequential Output------ --Sequential Input- --Random-
    Concurrency   1     -Per Chr- --Block-- -Rewrite- -Per Chr- --Block-- --Seeks--
    Machine        Size K/sec %CP K/sec %CP K/sec %CP K/sec %CP K/sec %CP  /sec %CP
    devback1        24G   521  99 192435  27 106363  15  2691  98 446441  27  1067  43
    Latency             16070us     454ms     174ms   14736us   98188us   30739us
    Version  1.96       ------Sequential Create------ --------Random Create--------
    devback1            -Create-- --Read--- -Delete-- -Create-- --Read--- -Delete--
              files  /sec %CP  /sec %CP  /sec %CP  /sec %CP  /sec %CP  /sec %CP
                 16 23295  46 +++++ +++ +++++ +++ +++++ +++ +++++ +++ +++++ +++
    Latency              5985us     563us     583us     960us      32us     259us
    1.96,1.96,devback1,1,1373282064,24G,,521,99,192435,27,106363,15,2691,98,446441,27,1067,43,16,,,,,23295,46,+++++,+++,+++++,+++,+++++,+++,+++++,+++,+++++,+++,16070us,454ms,174ms,14736us,98188us,30739us,5985us,563us,583us,960us,32us,259us

The CSV output can then be converted using:

    echo "1.96,1.96,devback1,1,1373282064,24G,,521,99,192435,27,106363,15,2691,98,446441,27,1067,43,16,,,,,23295,46,+++++,+++,+++++,+++,+++++,+++,+++++,+++,+++++,+++,16070us,454ms,174ms,14736us,98188us,30739us,5985us,563us,583us,960us,32us,259us" | bon_csv2html > bonnie_output.html

## Install an email server

Many of the monitoring tools will report back via email so it is necessary to have an email server installed.

The suggestion is to use the Postfix mail server as it is highly recommended - and to set Postfix to use Maildirs for storing the emails.

The tool procmail will be needed in the Postfix setup.

The command line tool 'mail' or similar will be needed when testing the sending of emails to a monitoring account.

| Disttribution | Versions |
| --- | --- |
| Debian | All versions |
| Ubuntu | All versions |

Install Postfix, Procmail and the 'mail' command with: 

    root@server1:~# apt-get install postfix procmail mailutils
   
NB - For default Debian this will remove the Exim email server.

When asked for set the server to be an internet server as this provides full services and set the FQDN to the FQDN of the server which was set up during the OS installation.

Then we will set all the accounts to use Maildir by default.  Edit /etc/postfix/main.cf and change the line beginning with mailbox_command to:

    mailbox_command = /usr/bin/procmail -a "$EXTENSION" DEFAULT=$HOME/Maildir/ MAILDIR=$HOME/Maildir

After making the change restart the Postfix server with:

    root@server1:~# postfix reload

What we would like to do is to redirect all mail for root to the server admin account.  This means we can log in with a safe account and read the server system emails.  It is also an idea to redirect all root emails to an email account which is used for monitoring the server.

Edit the file...

    /etc/aliases
	
...and add...

    root: root admin your.own.systems.email.address@example.com

...then enable the alias by running...

    root@server1:~# newaliases
	
Send a test email from the command line with:

    root@server1:~# mail root
	
And then fill in the fields.  Finish and send the email with Ctrl-D.  Check the email has been received and refer to

    /var/log/mail.log

for any error messages.

## Hardening the servers

### Removing insecure packages and services

There are some default packages which should be removed from standard Linux servers to harden the server.

| Disttribution | Versions |
| --- | --- |
| Debian | All versions |
| Ubuntu | All versions |

On Debian and derivatives the easiest way to remove insecure services and clients is to install two packages which carry out the task automatically.

    root@server1:~# apt-get install harden-servers harden-clients

Once these packages have been installed the admin user will be warned when they are going to install a package which is considered insecure.

### Set up the SSH server

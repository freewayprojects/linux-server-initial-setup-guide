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
* BIOS clock set correctly and server datetime set correct within a couple of minutes.
* SSH server installed and available
* Initial administration user created.  This user will be used to login to the server and is separate from the root user.
* Network address configured.
* The hostname, domain and DNS should be set up so that the server has its FQDN available via DNS.  This may just be available on a LAN.
* If necessary the reverse IP should be set up as certain services need to have the reverse IP working correctly.

Basically, the administrator should be able to login to the server over SSH to carry out this intial setup.  This is normally the point at which a hardware server has been put into a rack or located at its permanent location - or when a VM has been built and is available over a network.

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


    # screen -DR initialsetup


If the session is dropped then it can be picked up again with the same command.


    # screen -DR initialsetup


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
	
## Testing the RAM

### memtest86++

After the installation of the server operating system the mempry should have been tested using memtest86++.  This has the advantage of testing all of the RAM memory.

### memtester

It is also possible to test the RAM with memtester when logged in over SSH.  This is not able to test all of the memory but will at least provide a further check of the RAM.

| Disttribution | Versions |
| --- | --- |
| Debian | All versions |
| Ubuntu | All versions |

Install memtester with:

    root@server1:~# apt-get -s install memtester

and then run memtester with 

    root@server1:~# memtester 12 1


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

## Configure a time server client

Many of the services on a Linux server require the server datetime clock to be accurate.  To achieve this it is best to set up a client on the server which requests datetime updates from internet time servers.  For ease of use it is recommended that the tool chrony is used for this purpose.

| Disttribution | Versions |
| --- | --- |
| Debian | All versions |
| Ubuntu | All versions |

Install chrony with:

    root@server1:~# apt-get install chrony

After chrony has been installed edit the file

    /etc/chrony/chrony.conf

and set the server lines to be 

    server 0.pool.ntp.org
    server 1.pool.ntp.org
    server 2.pool.ntp.org

Then restart the chrony server with:

    root@server1:~# /etc/init.d/chrony restart

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

    root: admin your.own.systems.email.address@example.com

...then enable the alias by running...

    root@server1:~# newaliases
	
Send a test email from the command line with:

    root@server1:~# mail root
	
And then fill in the fields.  Finish and send the email with Ctrl-D.  Check the email has been received and refer to

    /var/log/mail.log

for any error messages.

## Hardening the servers

### Harden the SSH server

The SSH server can be made more secure by ensuring that the root user can not log in with a password but can only login using an SSH key.

Also, the accounts which can actually log in to the server can be restricted.

Before restricting the root login over SSH to be by key only it would be best to copy a key from the client machine to the server.  This can be carried out on the client by using ssh-copy-id.

    dev@workstation1:~$ ssh-copy-id root@server1.example.com

| Disttribution | Versions |
| --- | --- |
| Debian | All versions |

The following changes will restrict password logins to the admin account only and will restrict the root login to be via SSH key only.

    /etc/ssh/sshd_config

should be edited and the following lines edited/added:

    PermitRootLogin without-password
    AllowUsers root serveradmin

Also, the following lines should be added on most servers.  TCP or agent forwarding is unlikely to be needed and can be exploited even if a user account has been set to /bin/false or /usr/sbin/nologin.  Please note - do not necessarily add this to a LAMP development server which will be using xdebug as SSH forwarding may be needed in that case.

    AllowTcpForwarding no
    AllowAgentForwarding no

After editing sshd_config the the SSH server should be restarted with:

    root@server1:~# /etc/init.d/ssh restart

Administration of the server should be carried out by logging in as the admin user and using 'su -' to carry out root tasks.

| Disttribution | Versions |
| --- | --- |
| Ubuntu | All versions |

Ubuntu uses the sudo system and so the root account does not have the ability to log in to the server by default.  Administration of the server should be carried out by logging in as the admin user and using the sudo command to carry out root tasks.

### Get notifications about package updates

One of the main ways to keep a server secure is to ensure that the server packages are kept up to date.  One way to do this is to get the server to check the package statuses every day and to send out an email if packages need to be updated.

| Disttribution | Versions |
| --- | --- |
| Debian | All versions |
| Ubuntu | All versions |

    root@server1:~# apt-get install cron-apt

In the configuration file the MAILON configuration should be changed to 'upgrade' to make sure email is sent out when packages are available.

Edit the file 
    
    /etc/cron-apt/config

and change

    MAILON="error"
	
to

    MAILON="upgrade"

### Alternatively - enable automatice security updates

For non-critical servers it may be best to enable automatic updates.

| Disttribution | Versions |
| --- | --- |
| Debian | All versions |
| Ubuntu | All versions |

    root@server1:~# apt-get install unattended-upgrades apt-listchanges
    root@server1:~# dpkg-reconfigure -plow unattended-upgrades

Then edit

    /etc/apt/apt.conf.d/50unattended-upgrades
   
 and set 
 
    Unattended-Upgrade::Mail "root";
    Unattended-Upgrade::Automatic-Reboot "true";

### Monitor the log files

The log files on the server should be regularly monitored so problems can be identified.  It is best if an agregation system can be used to summarise the log files for easier interpretation.

| Disttribution | Versions |
| --- | --- |
| Debian | All versions |
| Ubuntu | All versions |

    root@server1:~# apt-get install logwatch

By default logwatch is configured to send daily emails to root which contain a summary of the log files.

### Block malicious attempts to access the server

Attempts to break-in to the server should be monitored and blocked.  This should mainly monitor attempts to log in via SSH but other services should be monitored as well.

There are several tools available for carrying out this task:

* Fail2ban
* Denyhosts
* SSHguard

| Disttribution | Versions |
| --- | --- |
| Debian | All versions |
| Ubuntu | All versions |

The tool Fail2ban works well on Debian and Ubuntu and the default configuration works well.

    root@server1:~# apt-get install fail2ban

### Install tools to monitor for rootkits

If a server has had a rootkit been rooted then this is bad news and the server can be considered compromised.  Notification of the rootkit is important however as the system administrator cna take steps to examine the intrusion and transfer services to another server.

There are serveral rootkit monitoring tools available including:

* chkrootkit
* rkhunter

| Disttribution | Versions |
| --- | --- |
| Debian | All versions |
| Ubuntu | All versions |

Install the tool chkrootkit with:

    root@server1:~# apt-get install chkrootkit

Then in the configuration file

    /etc/chkrootkit.conf

set 

    RUN_DAILY="true"
	
Install rkhunter with:

    root@server1:~# apt-get install rkhunter

The default install of rkhunter is set to report issues daily.

### Install monitoring of configuration files.

It is important to monitor any changes which are made to the configuration files.  There are many tools which carry out this function.  The tool etckeeper is one of the easiest to set up and puts all the configuration files under version control.

| Disttribution | Versions |
| --- | --- |
| Debian | All versions |

On Debian etckeeper is set to use git by default - even if git is not installed.  So, to install etckeeper run:

    root@server1:~# apt-get install etckeeper git-core

Then changes to any configuration files can be tracked with:

    root@server1:~# git --work-tree=/etc status
    root@server1:~# git --work-tree=/etc log    

| Disttribution | Versions |
| --- | --- |
| Ubuntu | All versions |

On Ubuntu the versioning system Bazaar is used by default.  The repository can be switched to using git if necessary.

    root@server1:~# apt-get install etckeeper

Then changes to any configuration files can be tracked with:

    root@server1:~# bzr status /etc
    root@server1:~# bzr log /etc

### Install administration tools

There are many tools used by administrators when looking after a server.  Here is a basic list of tools which should be made available by default on Linux servers:

* Midnight Commander - file manager
* Emacs - Professional level editor
* Vim - Editor
* locate - tool for fast searching for files

| Disttribution | Versions |
| --- | --- |
| Debian | All versions |

    root@server1:~# apt-get install mc emacs vim locate

| Disttribution | Versions |
| --- | --- |
| Ubuntu | All versions |

On Ubuntu Vim is installed during the default installation.

    root@server1:~# apt-get install mc emacs locate

# Installing LAMP server

| Disttribution | Versions |
| --- | --- |
| Debian | All versions |

LAMP packages were installed by running:

    root@server1:~# apt-get install apache2-mpm-itk mysql-server php5 php5-mysql
    root@server1:~# apt-get install php5-intl php5-curl php5-gd php5-mcrypt php5-cli
    root@server1:~# apt-get install php-apc
    root@server1:~# a2enmod rewrite
    root@server1:~# apache2ctl restart

MySQL was set up to use the my-huge.cnf file.  This was then edited.

In the [mysqld] sections the line:

    expire_logs_days        = 1

was added to the [mysqld] section.

# Munin install

Munin is a great way to see the server perfomance and resource usage over time.  If a LAMP server has been set up then Munin is a very useful tool to have.

| Disttribution | Versions |
| --- | --- |
| Debian | 8 |

    root@server1:~# apt-get install libcgi-fast-perl libapache2-mod-fcgid
    root@server1:~# apt-get install munin

In the Apache configuration file we want to allow external access to the Munin graphs - but for an authorised user only.

Set up a pasword file with:

    # htpasswd -c /etc/munin/munin-htpasswd munin

Then in /etc/munin/apache24.conf

Add in:

        # Require local                                                                           
        AuthType Basic
        AuthName "Protected area"
        AuthUserFile /etc/munin/munin-htpasswd
        Require valid-user

Some extra plugins should be enabled for Munin to check other services - specifically Apache and MySQL.

We need a couple of extra packages.

This will enhance the display of the graphs.

    # apt-get install libcgi-fast-perl libapache2-mod-fcgid

NB- This does not seem to be working correctly yet.

This will enable the MySQL plugins:

    # apt-get install libcache-cache-perl
    # ln -s /usr/share/munin/plugins/mysql_* /etc/munin/plugins

Actually - to get all MySQL plugins working use:

    # munin-node-configure --shell | sh

This will get the Apache plugins working:

    # apt-get install libwww-perl 
    # ln -s /usr/share/munin/plugins/apache_* /etc/munin/plugins

To monitor the hard drives:

    # apt-get install smartmontools
    # munin-node-configure --shell | sh

# Monitoring the hard drives

If Smartmontools is installed then we can have it regularly testing the hard drives.

| Disttribution | Versions |
| --- | --- |
| Debian | 8 |

In /etc/default/smartmontools set

    start_smartd=yes
    
    

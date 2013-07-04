linux-server-initial-setup-guide
================================

# Overview

This guide is designed to be used after the initial installation of Linux has been completed.  The installation of Linux can be carried out using the [Linux Server Installation Guide](https://github.com/freewayprojects/linux-server-installation-guide).

So, this guide should be used to pick up from where [Linux Server Installation Guide](https://github.com/freewayprojects/linux-server-installation-guide) finished.

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

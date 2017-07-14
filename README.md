# Linux: Active Directory Integration

This is a brief to demo for joining a CentOS/RHEL 6 or 7 server to Active Directory.  I am using Ansible to perform the automation of these tasks, but we can break this down to see what changes are occuring.  One point that I want to make is that this is not a full fleshed out solution.

### Automation Steps

  - Installing required packages
    - In this step we install several packages that are required in order to join a Linux server to AD.

  - Join Server to domain
    - In this step we use the `adcli` tool in order to add this server to the domain
    - We also copy a modified krb5.conf to /etc/krb5.conf

  - Configure SSSD
    - Copy a modified sssd.conf file to /etc/sssd/sssd.conf
    - Update PAM modules using authconfig
    - Enable the SSSD service
    - Enable the oddjobd service

### Post Automation

At this point, we are done. The host is now domain joined and will authenticate users in AD to Linux! There are still some follow-up steps that should be done (easily automated as well), and we'll look at that here.


### What is really happening?

Below is what an admin would do manually. You will need to adjust the variables to work for you, obviously.

```
stuff
```

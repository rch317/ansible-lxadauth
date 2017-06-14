# ansible-lxadauth
Ansible role to configure CentOS/RHEL hosts to join an Active Directory domain for authentication.

Tested against CentOS 7.x, with plans to test against CentOS 6.x and both RHEL 6/7.  Eventually this will also work with the more popular Debian flavors as well.

This role was developed for my needs. Feel free to use it as you see fit. If you see a better way, please let me know. I'm coming from a SaltStack background, so Ansible is fairly new to me. My
company is using Ansible, else this would be done in SaltStack...

;)

# Requirements
A fully functional and properly configured Active Directory domain, that is reachable by the hosts you are intending to configure. You should also configure all hosts in the domain to use a common NTP server.

You will need user credentials, with the correct permissions, to add to your domain.

# Vars

At some point I'll add to this, things like the ad_access_filter and such. Right now this is working fine for what I'm doing right now.

* `adauth_user` - User with permissions to add to the domain
* `adauth_pass` - Password for the AD domain user.
* `adauth_server_ou` - The OU to put the host(s) in.
* `adauth_domain` - The domain you are joining

# ansible-lxadauth
Ansible role to configure CentOS/RHEL hosts to join an Active Directory domain for authentication.

### Requirements
  - Functional Active Directory Domain
  - NTP Configured on all hosts
  - Domain User with ability to add objects to the domain

### Vars

Currently the variables are pretty simple. Going forward you would want to extend the configuration for 'sssd' and create variables for those values. For instance, `ad_access_filter` or locking down access to machines by narrowing the OU being searched. This can drastically increase performance when dealing with a high number of objects in a domain.

* `adauth_user` - User with permissions to add to the domain
* `adauth_pass` - Password for the AD domain user.
* `adauth_server_ou` - The OU to put the host(s) in.
* `adauth_domain` - The domain you are joining

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

1. Create an advars.env file
```
cat <<EOF>> /root/advars.env
#!/bin/sh

### Set your domain
export adauth_domain=ISLAND.LOCAL

### Set the OU of this host will be added to
adauth_server_ou="DC=ISLAND,DC=LOCAL"

### Set a user who has access to add objects to the domain
export adauth_user="lxadjoin"

### Set the password for the user above.
export adauth_pass="cya+SdhfWZBUze+q"

EOF

### Make sure only we can read this file
chmod 0600 /root/advars.env

### source our variables file
source /root/advars.env

### Validate that our variables exported correctly
set | grep ^adauth.*$
```

2. Install the required packages:
```
yum install -y epel-release \
    libselinux-python \
    adcli \
    oddjob \
    oddjob-mkhomedir \
    sssd-client \
    sssd-ad \
    sssd-krb5 \
    sssd-krb5-common \
    krb5-workstation
```
3. Join the server to the domain:
  ```
echo -n ${adauth_pass} | \
  /usr/sbin/adcli join --stdin-password -O ${adauth_server_ou} \
  -U ${adauth_user} -D ${adauth_domain} -H $(hostname -f) \
  --user-principal=host/$(hostname -f)@${adauth_domain^^}

### Backup existing krb5.conf
cp /etc/krb5.conf{,.$(date +%s)}  

### Create new krb5.conf
cat <<EOF>/etc/krb5.conf

[libdefaults]
    default_realm = ${adauth_domain^^}
    default_keytab_name = /etc/krb5.keytab
    dns_lookup_kdc = true

[domain_realm]
    .${adauth_domain,,} = ${adauth_domain^^}  
    ${adauth_domain,,} = ${adauth_domain^^}

[logging]
    default = FILE:/var/log/krb5libs.log
    kdc = FILE:/var/log/krb5kdc.log
    admin_server = FILE:/var/log/kadmind.log

EOF

 ### set permissions on /etc/krb5.conf
 chmod 0644 /etc/krb5.conf
```

4. Configure SSSD
```
cat <<EOF>>/etc/sssd/sssd.conf
[nss]
filter_groups = root
filter_users = root
reconnection_retries = 3

[pam]
reconnection_retries = 3

[ssh]

[sssd]
config_file_version = 2
reconnection_retries = 3
sbus_timeout = 30
services = nss, pam, ssh
dns_discovery_domain = ${adauth_domain,,}
domains = ${adauth_domain^^}
debug_level = 0x0150

[domain/${adauth_domain^^}]
debug_level = 0x0150
enumerate = false
cache_credentials = false

# set providers
id_provider = ad
access_provider = ad
auth_provider = ad

# need to set the realm for krb auth
krb5_realm = ${adauth_domain^^}

# ad info
ad_domain = ${adauth_domain,,}
ad_hostname = $(hostname -f)

# set dns update refresh interval in seconds
dyndns_refresh_interval = 14400

# enabled by default in 1.13.4, which causes us issues
ad_gpo_access_control = disabled

# changing how the id mapping works for uid/gid since those aren't in ad
ldap_id_mapping = true
ldap_schema = ad
override_homedir = /home/%u
default_shell = /bin/bash
ldap_user_shell = loginShell

# pull in public key from ad
ldap_user_ssh_public_key = altSecurityIdentities

EOF

```
5. Configure PAM modules
```
authconfig --enablesssd --enablesssdauth \
 --disableldap --disableldapauth --disablekrb5 --enablemkhomedir --update
```

6. Enable the SSSD service & Make sure it has started
```
systemctl enable sssd
systemctl restart sssd
```

7. Enable the oddjobd service & make sure it is started
```
systemctl enable oddjobd
systemctl restart oddjobd
```

8. At this point the server should be joined to the domain.  Test an AD account to validate. If you experience problems, try restarting the server and trying again. Some common trouble shooting issues:

    - Make sure NTP is enabled and date/time synchronized between client machine and AD machine.
    - Make sure no typos are in your config files.
    - DNS is properly configured for AD

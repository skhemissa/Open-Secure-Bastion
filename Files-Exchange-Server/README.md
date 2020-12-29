# Files-Exchange-Server

Update then install ldap components.
```
$ sudo apt update && \
 sudo apt upgrade -y && \
 sudo apt install libnss-ldap libpam-ldap ldap-utils --no-install-recommends -y
```

In dpkg configure screen (all items are based on the following project [Secure OpenLDAP Debian](https://github.com/skhemissa/Secure-OpenLDAP-Debian)).
* LDAP server Uniform Resource Identifier: ldap://<ldap_ip>
* Distinguished name of the search base: test=srv,dc=local
* LDAP version to use: 3
* LDAP administrative account: cn=admin,dc=test,dc=local (other account to be considered)
* LDAP root account password: <admin_password>
* Also, before removing this package, it is wise to remove the "ldap" entries from nsswitch.conf to keep basic services functioning: Ok
* Allow LDAP admin account to behave like local root?: Ok
* Does the LDAP database require login?: No
* LDAP administrative account: cn=admin,dc=test,dc=local
* LDAP administrative password: <admin_password>

Finalize configuration.
```
$ sudo vi /etc/nsswitch.conf # replace lines 7,8 and 9 with the following 
passwd:         compat systemd ldap
group:          compat systemd ldap
shadow:         compat

root@www:~# vi /etc/pam.d/common-password # replace the line 26 with the following
password        [success=1 user_unknown=ignore default=die]     pam_ldap.so try_first_pass

root@www:~# vi /etc/pam.d/common-session # add to the end of the file
session optional        pam_mkhomedir.so skel=/etc/skel umask=077

$ sudo /sbin/reboot
```
Install clamav then update virus database
```
$ sudo apt install clamav clamav-freshclam clamav-daemon --no-install-recommends -y && \
 sudo systemctl stop clamav-freshclam && \
 sudo freshclam && \
 sudo systemctl start clamav-freshclam && \
 sudo systemctl enable clamav-daemon && \
 sudo systemctl restart clamav-daemon
```

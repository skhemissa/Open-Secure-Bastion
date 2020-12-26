# Open-Secure-Bastion

All components work with the following LDAP configuration [Secure OpenLDAP server on Debian 10 Buster](https://github.com/skhemissa/Secure-OpenLDAP-Debian)

ldap.ldif provides a minimal structure for the LDAP directory including three users and two groups.

Provided groups are used in the configuration of each service hosted in the Open Secure Bastion:
* Members of "web1,ou=groups,dc=test,dc=local" are allowed for accessing a backend web application named web1 protected by the HTTPs Reverse proxy;
* Members of "cn=guacamole,ou=groups,dc=test,dc=local" are allowed for accessing the Remote Access Gateway.
... users could be members of multiple groups.

Don't forget to generate a password for each accout.
```
$ /sbin/slappasswd
New password:
Re-enter new password:
{SSHA}<password>
```
Then import the structure.
```
$ sudo ldapadd -x -H ldaps://localhost -D cn=admin,dc=test,dc=local -W -f ldap.ldif
```

## HTTPs Reverse proxy
Based on Apache2 with mod_proxy... and several other modules.
For protecting:
1. Internal web applications 
2. The Remote Access Gateway for accessing remote servers.

Authentication is done using "Basic Authentication" that's bind on a OpenLDAP Server: user & password + group membership is required.

One group is assigned to a protected application or a group of applications with same users.

## Remote Access Gateway
Based on Guacamole Apache.
For accessing to remote serveur using SSH, RDP and VNC;

Identification is managed by reading the value of REMOTE_USER filed from the http header.

The http header is enriched by the HTTPs Reverse proxy based on authentication.

The user is automatically created in the Remote Access Gateway: connections and/or groups assignments are done by a Remote Access Gateway administrator.

File exchange between Remote Access Gateway and remote servers is blocked.

>***The trafic to the Remote Access Gateway must be stricly filtered. It must be allowed only from the HTTPS reverse Proxy. Otherwise, both components must be hosted in a dedicated DMZ.***

## Files Exchange Server with real-time antivirus
Based on sftp on a debian machine and ClamAV antivirus.

Files are exchanged using the Remote Access Gateway thanks to file donwload / upload features.

Mounts could be done on remote server using sshfs available for main Linux flavours and [SSHFS-Win](https://github.com/billziss-gh/sshfs-win) for Windows hosts.

To be considered for fulture releases: cifs and nfs.

Authentication is also binded to the OpenLDAP server.

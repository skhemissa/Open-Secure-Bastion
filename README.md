# Open-Secure-Bastion

All components work with the following LDAP configuration [Secure OpenLDAP server on Debian 10 Buster](https://github.com/skhemissa/Secure-OpenLDAP-Debian)

## HTTPs Reverse proxy
For protection  :
1. Internal web applications 
2. The Remote Access Gateway for accessing remote servers.

Authentication is done using "Basic Authentication" that's bind on a OpenLDAP Server: user & password + group membership (one group per protected application)

## Remote Access Gateway using Guacamole Apache:
1. Supported protocols: SSH, RDP and VNC;
2. File exchange with remote servers is blocked.

Identification is managed by reading the value of REMOTE_USER filed from the http header.

The http header is enriched by the HTTPs Reverse proxy based on authentication.

The user is automatically created in the Remote Access Gateway: connections and/or groups assignments are done by a Remote Access Gateway administrator.

>***The trafic to the Remote Access Gateway must be stricly filtered. It must be allowed only from the HTTPS reverse Proxy. Otherwise, both components must be hosted in a dedicated DMZ.***

## Files Exchange Server with real-time antivirus
Files are exchanged using the Remote Access Gateway thanks to file donwload / upload features.
Authentication is also binded to the OpenLDAP server.

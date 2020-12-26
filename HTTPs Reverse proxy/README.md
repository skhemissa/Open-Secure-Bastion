# HTTPs Reverse proxy
Work on Debian 10 (Buster)

Update the system then install apache2.
```
$ sudo apt update && \ 
 sudo apt upgrade -y && \
 sudo install apache2 -y --no-install-recommends
```
Activate required apache2 modules then desactivate default site.

Modules headers, rewrite and proxy_wstunnel are required for protecting the Remote Access Gateway.
```
$ sudo a2enmod authnz_ldap proxy_http  ssl  headers rewrite proxy_wstunnel && \
 sudo a2dissite 000-default 
```
Disable port 80 listening for apache2 and comment the line Listen 80.
```
$ sudo vi /etc/apache2/ports.conf
#Listen 80
```
Upload the private RSA key file in the directory /etc/ssl/private/.

Upload the ssl certifcate file in the directory /etc/ssl/certs/.

Otherwise, create a self-signed certificate.
```
$ sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/secure_reverse.key -out /etc/ssl/certs/secure_reverse.crt
Country Name (2 letter code) []:
State or Province Name (full name) []:
Locality Name (eg, city) []:
Organization Name (eg, company) []:
Organizational Unit Name (eg, section) []:
Common Name (eg, fully qualified host name) []: reverse.test.local
Email Address []:

```
Activate perfect forward secrecy, which generates ephemeral session keys.

This action is going to take a very very very very very very very long time.
```
$ sudo openssl dhparam -out /etc/ssl/certs/dhparam.pem 4096
```
Copy the file reverse_https.conf located in the directory conf of this project in the directory /etc/apache2/sites-enabled/ in the reverse server.

Update SSLCertificateFile and SSLCertificateKeyFile paths if you are using certificate files with a specific naming convention.

Update ServerName.

There are two backend web applications configured:
* Location /guacamole/ = Remote Access Gateway with specific configuration: authorization is allowed if the user is authentication and member of the group "cn=guacamole,ou=groups,dc=test,dc=local". The username of authenticated user is added to the header for single sing-on on the Remote Access Gateway. You have to:
  * Update ProxyPass and ProxyPassReverset with the url of the Remote Access Gateway ; 
  * Update LDAP URL, the username and the password used to bind to the LDAP server.
* Location /web1/ = backend web server with generic configuration that could be used for other web server (you have to check with the owner of the backend web application for additional parameters to be forwarded): authorization is allowed if the user is authentication and member of the group "cn=web1,ou=groups,dc=test,dc=local". You have to:
  * Provide a name for the location that will be reached with the following URL : https://<reverse-proxy>/location/
  * Update ProxyPass and ProxyPassReverse  with the url of the backend web application ;
  * Update AuthLDAPURL, AuthLDAPBindDN and AuthLDAPBindPassword used to bind to the LDAP server. A specific LDAP server could be configured for each backend web application.
  * Create a new group in LDAP directory for each application or reuse existing group if users are the same;
  * If you are using subgroups, uncomment AuthLDAPMaxSubGroupDepth and set depth of used sub-groups to validate the group membership of the authenticated user.
  * For user authentication only, uncomment "Require valid-use" and comment AuthLDAPGroupAttribute and "Require ldap-group";
  * Duplicate this location for additional protected backend web applications.

Activate the new site then restart apache2.
```
$ sudo a2ensite reverse_https && \
 sudo systemctl restart apache2
```
... et voilà

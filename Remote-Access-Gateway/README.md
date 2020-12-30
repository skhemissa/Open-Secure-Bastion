# Remote Access Gateway

The Remote Access Gateway is based on [Apache Guacamole](https://guacamole.apache.org/).

It is a gateway that allows accessing remote servers using standard protocols like VNC, RDP, and SSH through a browser.

Tomcat and Postgresql are required.

Please note that the authentication is achieved by the HTTPs Reverse Proxy that send the username to the Remote Access Gateway in the header (field "REMOTE_USER").

The user is automatically created in the Remote Access Gateway: connections and/or groups assignments are done by a Remote Access Gateway administrator.

>***The trafic to the Remote Access Gateway must be stricly filtered. It must be allowed only from the HTTPS reverse Proxy. Otherwise, both components must be hosted in a dedicated DMZ.***

## Download Apache Guacamole components
Download the following components from [Apache Guacamole release page](https://guacamole.apache.org/releases/).
* Source code: guacamole-server-*.tar.gz
* Extensions:
  * guacamole-auth-jdbc-*.tar.gz
  * guacamole-auth-header-*.tar.gz

Download the last pg java driver from https://jdbc.postgresql.org/download.html#current
```
$ cd /tmp && \
 wget https://jdbc.postgresql.org/download/postgresql-42.2.18.jar
```

Put the archive in /tmp then uncompress it.
```
$ cd /tmp && \
 tar -xzf guacamole-server-1.2.0.tar.gz && \
 tar zxvf guacamole-auth-jdbc-1.2.0.tar.gz && \
 tar zxvf guacamole-auth-header-1.2.0.tar.gz
```
## Tomcat installation and configuration
Prerequisites installation
```
$ sudo apt install default-jdk -y --no-install-recommends && \
 sudo groupadd tomcat && \
 sudo useradd -s /bin/false -g tomcat -d /opt/tomcat tomcat
```
Download the latest version of tomcat from https://tomcat.apache.org/download-90.cgi then configure it.
```
$ cd /tmp && \
 wget https://mirror.ibcp.fr/pub/apache/tomcat/tomcat-9/v9.0.41/bin/apache-tomcat-9.0.41.tar.gz && \
 sudo mkdir /opt/tomcat && \
 sudo tar xzvf apache-tomcat-9*tar.gz -C /opt/tomcat --strip-components=1 && \
 cd /opt/tomcat && \
 sudo chgrp -R tomcat /opt/tomcat && \
 sudo chmod -R g+r conf && \
 sudo chmod g+x conf && \
 sudo chown -R tomcat webapps/ work/ temp/ logs/
```
Get the JAVA_HOME: it is the third column from the output of the command below.
```
$ sudo update-java-alternatives -l
java-1.11.0-openjdk-amd64      1111       /usr/lib/jvm/java-1.11.0-openjdk-amd64
```
Add Tomcat to systemd (Adjust JAVA_HOME according to the previous command).
```
$ sudo vi /etc/systemd/system/tomcat.service
[Unit]
Description=Apache Tomcat Web Application Container
After=network.target

[Service]
Type=forking

Environment=JAVA_HOME=/usr/lib/jvm/java-1.11.0-openjdk-amd64
Environment=CATALINA_PID=/opt/tomcat/temp/tomcat.pid
Environment=CATALINA_HOME=/opt/tomcat
Environment=CATALINA_BASE=/opt/tomcat
Environment='CATALINA_OPTS=-Xms512M -Xmx1024M -server -XX:+UseParallelGC'
Environment='JAVA_OPTS=-Djava.awt.headless=true -Djava.security.egd=file:/dev/./urandom'

ExecStart=/opt/tomcat/bin/startup.sh
ExecStop=/opt/tomcat/bin/shutdown.sh

User=tomcat
Group=tomcat
UMask=0007
RestartSec=10
Restart=always

[Install]
WantedBy=multi-user.target

$ sudo systemctl daemon-reload && \
 sudo systemctl enable tomcat && \
 sudo systemctl restart tomcat
```

## Postgresql installation

```
$ sudo apt update && \
 sudo apt upgrade -y && \
 sudo apt install postgresql logrotate -y --no-install-recommends
```

Configure a database for Apache Guacamole then create an account for accessing this database.
```
$ sudo -u postgres createdb guacamole_db && \
 sudo -u postgres sh -c "cat /tmp/guacamole-auth-jdbc-1.2.0/postgresql/schema/*.sql | psql -d guacamole_db -f -" && \
 sudo -u postgres -H -- psql -d guacamole_db -c "CREATE USER guacamole_user WITH PASSWORD 'some_password'" && \
 sudo -u postgres -H -- psql -d guacamole_db -c "GRANT SELECT,INSERT,UPDATE,DELETE ON ALL TABLES IN SCHEMA public TO guacamole_user" && \
 sudo -u postgres -H -- psql -d guacamole_db -c "GRANT SELECT,USAGE ON ALL SEQUENCES IN SCHEMA public TO guacamole_user"
```
## Apache Guacamole configuration
Prerequisites installation
```
$ sudo apt install libcairo2-dev libjpeg62-turbo-dev libpng-dev libtool-bin libossp-uuid-dev freerdp2-dev libpango1.0-dev libssh2-1-dev libvncserver-dev libssl-dev libwebp-dev make -y
```
Create guacd, the server-side proxy used by the Apache Guacamole web application. 
```
$ cd /tmp/guacamole-server-1.2.0  && \
 ./configure --with-init-dir=/etc/init.d  && \
 make && \
 sudo make install && \
 sudo ldconfig && \
 sudo systemctl enable guacd && \
 sudo systemctl restart guacd
```
Deploy and configure the Apache Guacamole web applixation
```
$ cd /tmp  && \
 mv guacamole-1.2.0.war guacamole.war && \
 sudo cp guacamole.war /opt/tomcat/webapps/ && \
 sudo mkdir /etc/guacamole && \
 cd /etc/guacamole && \
 sudo touch guacamole.properties && \
 sudo touch user-mapping.xml
```
Configure Apache Guacamole web applixation for accessing guacd, database access and users auto enrollment
```
$ sudo vi guacamole.properties
guacd-hostname: localhost
guacd-port: 4822

# PostgreSQL properties
postgresql-hostname: localhost
postgresql-port: 5432
postgresql-database: guacamole_db
postgresql-username: guacamole_user
postgresql-password: some_password

# Auto-creating database users
postgresql-auto-create-accounts: true
```
Copy Apache Guacamole components and pg java driver.
```
$ sudo mkdir /etc/guacamole/extensions/ && \
 sudo cp /tmp/guacamole-auth-jdbc-1.2.0/postgresql/guacamole-auth-jdbc-postgresql-1.2.0.jar /etc/guacamole/extensions/ && \
 sudo cp /tmp/guacamole-auth-header-1.2.0/guacamole-auth-header-1.2.0.jar /etc/guacamole/extensions/ && \
 sudo mkdir /etc/guacamole/lib && \
 sudo cp /tmp/postgresql-42.2.18.jar /etc/guacamole/lib && \
 sudo systemctl restart tomcat
```

... et voilà.
You can access Apache Guacamole protected by the proxy with the following URM : https://<reverse_proxy>/guacamole

# How to install Liferay 7.x on Debian 10

[TOC levels=1-4]

## Pre-Checks

1. Check debian version that be true.
```shell
lsb_release -a

```
2. Check DNS settings

    It is better if you use the [Shecan](https://shecan.ir/) DNS servers.

> Edit below and add new name servers here and comment out existing name servers:

```shell
 nano /etc/resolve.conf
```

3. Update the system

```
apt-get update
apt update
apt upgrade
```

4. Install some utility packages:
```shell
apt install screen wget git unzip  -y

```

## Install The Database : Postgresql
* We install default `postgresql-11` version on Debian 10.

```bash
root@deb-liferay:~# apt install postgresql -y
```

> Enable the service

```bash
root@deb-liferay:~# systemctl enable postgresql
Synchronizing state of postgresql.service with SysV service script with /lib/systemd/systemd-sysv-install.
Executing: /lib/systemd/systemd-sysv-install enable postgresql
```
> Check the service
```shell
root@deb-liferay:~# systemctl status postgresql
● postgresql.service - PostgreSQL RDBMS
   Loaded: loaded (/lib/systemd/system/postgresql.service; enabled; vendor preset: enabled)
   Active: inactive (dead) since Wed 2021-02-17 12:35:34 +0330; 1h 49min ago
 Main PID: 3116 (code=exited, status=0/SUCCESS)

Feb 17 12:29:48 deb-liferay systemd[1]: Starting PostgreSQL RDBMS...
Feb 17 12:29:48 deb-liferay systemd[1]: Started PostgreSQL RDBMS.
Feb 17 12:35:34 deb-liferay systemd[1]: postgresql.service: Succeeded.
Feb 17 12:35:34 deb-liferay systemd[1]: Stopped PostgreSQL RDBMS.
```
> Change the data directory

```shell
root@deb-liferay:~# mkdir -p /home/data/postgres
root@deb-liferay:~# cp -Rf /var/lib/postgresql/11/main /home/data/postgres/
root@deb-liferay:~# mv /var/lib/postgresql/11/main /var/lib/postgresql/11/main.back
root@deb-liferay:~# chown -Rf postgres:postgres /home/data/postgres
root@deb-liferay:~# chmod 0700 /home/data/postgres
```
- Edit postgres configuration file (/etc/postgresql/11/main/postgresql.conf) and change default `data_directory` by adding bellow line at end of file after `# Add settings for extensions here` line like

```shell
#------------------------------------------------------------------------------
# CUSTOMIZED OPTIONS
#------------------------------------------------------------------------------

# Add settings for extensions here
data_directory = '/home/data/postgres'
```

> Then restart the `postgresql` service

```shell
root@deb-liferay:~# systemctl restart postgresql.service
```

* Create an user and database for `liferay`
> Create User
```shell
root@deb-liferay:~# su postgres
postgres@deb-liferay:/root$ createuser --interactive --pwprompt
could not change directory to "/root": Permission denied
Enter name of role to add: liferay
Enter password for new role:
Enter it again:
Shall the new role be a superuser? (y/n) n
Shall the new role be allowed to create databases? (y/n) n
Shall the new role be allowed to create more new roles? (y/n) n
```

> Create database
```
postgres@deb-liferay:/root$ createdb -O liferay  lportal735
```
| **Note:** The `lportal735` means `Liferay Portal 7.3.5`, Please choose right name as version of lferay that you wanna to install it.
>Test login

```shell
root@deb-liferay:~# psql -h localhost  -U liferay lportal735
Password for user liferay:
psql (11.10 (Debian 11.10-0+deb10u1))
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, bits: 256, compression: off)
Type "help" for help.

lportal735=> \q
```

## Install AdoptedOpenJDK 8

### Pre-requisites

First check current installation of `java` on this machine by running below command, if any version of java installed please remove this.

```shell
root@deb-liferay:~# java -version
-bash: java: command not found
```
```shell
root@deb-liferay:~# apt install -y wget gnupg software-properties-common
root@deb-liferay:~# wget -qO - https://adoptopenjdk.jfrog.io/adoptopenjdk/api/gpg/key/public |  apt-key add -
OK
root@deb-liferay:~# sudo add-apt-repository --yes https://adoptopenjdk.jfrog.io/adoptopenjdk/deb/
-bash: sudo: command not found
root@deb-liferay:~# add-apt-repository --yes https://adoptopenjdk.jfrog.io/adoptopenjdk/deb/
```

## Install JDK 8
```shell
root@deb-liferay:~# apt update -y
root@deb-liferay:~# apt install adoptopenjdk-8-hotspot -y
root@deb-liferay:~# update-alternatives --config java
There are 3 choices for the alternative java (providing /usr/bin/java).

  Selection    Path                                                Priority   Status
------------------------------------------------------------
  0            /usr/lib/jvm/java-11-openjdk-amd64/bin/java          1111      auto mode
* 1            /usr/lib/jvm/adoptopenjdk-8-hotspot-amd64/bin/java   1081      manual mode
  2            /usr/lib/jvm/java-11-openjdk-amd64/bin/java          1111      manual mode
  3            /usr/lib/jvm/java-7-openjdk-amd64/jre/bin/java       1071      manual mode

Press <enter> to keep the current choice[*], or type selection number: 1

root@deb-liferay:~# java -version
openjdk version "1.8.0_282"
OpenJDK Runtime Environment (AdoptOpenJDK)(build 1.8.0_282-b08)
OpenJDK 64-Bit Server VM (AdoptOpenJDK)(build 25.282-b08, mixed mode)
root@kosar:~# update-alternatives --config javac
There are 2 choices for the alternative javac (providing /usr/bin/javac).

  Selection    Path                                                 Priority   Status
------------------------------------------------------------
* 0            /usr/lib/jvm/java-11-openjdk-amd64/bin/javac          1111      auto mode
  1            /usr/lib/jvm/adoptopenjdk-8-hotspot-amd64/bin/javac   1081      manual mode
  2            /usr/lib/jvm/java-11-openjdk-amd64/bin/javac          1111      manual mode

Press <enter> to keep the current choice[*], or type selection number: 1
update-alternatives: using /usr/lib/jvm/adoptopenjdk-8-hotspot-amd64/bin/javac to provide /usr/bin/javac (javac) in manual mode

root@deb-liferay:~# javac -version
javac 1.8.0_282
root@deb-liferay:~#
```

## Set Java Home Environment
```shell
root@deb-liferay:~# update-alternatives --config java
There is only one alternative in link group java (providing /usr/bin/java): /usr/lib/jvm/adoptopenjdk-8-hotspot-amd64/bin/java
Nothing to configure.
```

## Installing Liferay 7.x

Find right version from this locations :
* [Sourceforge](https://sourceforge.net/projects/lportal/files/Liferay%20Portal/)
* [Github](https://github.com/liferay/liferay-portal/releases)
* [Lifery CDN](https://cdn.lfrs.sl/releases.liferay.com/portal/)

### Using `screen` and download it with `wget`

```shell
root@deb-liferay:~# screen -R -D liferay
root@deb-liferay:~# wget https://cdn.lfrs.sl/releases.liferay.com/portal/7.3.5-ga6/liferay-ce-portal-tomcat-7.3.5-ga6-20200930172312275.tar.gz
```
| **Tip** you can use `CTRL+A`+`CTRL-D` to detach screen and again run `screen -R -D liferay` to return to this session.

### Extract and Installing liferay

```shell
root@deb-liferay:~# tar xzf liferay-ce-portal-tomcat-7.3.5-ga6-20200930172312275.tar.gz -C /opt/
root@deb-liferay:~# ln -s /opt/liferay-ce-portal-7.3.5-ga6 /opt/liferay
root@deb-liferay:~# ln -s /opt/liferay/tomcat-9.0.37 /opt/liferay/tomcat
```

### Start liferay and check log files

```shell
root@deb-liferay:~# /opt/liferay/tomcat/bin/startup.sh
Using CATALINA_BASE:   /opt/liferay/tomcat
Using CATALINA_HOME:   /opt/liferay/tomcat
Using CATALINA_TMPDIR: /opt/liferay/tomcat/temp
Using JRE_HOME:        /usr/lib/jvm/adoptopenjdk-8-hotspot-amd64
Using CLASSPATH:       /opt/liferay/tomcat/bin/bootstrap.jar:/opt/liferay/tomcat/bin/tomcat-juli.jar
Tomcat started.

| **Note** JRE_HOME must assigned to jdk8. if it is jdk11 please edit (/etc/profile) file and add this line to end of file :
| export JAVA_HOME="/usr/lib/jvm/adoptopenjdk-8-hotspot-amd64"
| export PATH=$JAVA_HOME/bin:$PATH

root@deb-liferay:~# tail -f /opt/liferay/tomcat/logs/catalina.out
```
> Now, Go to http://SERVER-IP:8080 and finish setup of Liferay Portal.
![](../images/setup-liferay.png)



### Install systemd daemon
* Reset all settings in `setenv.sh` file
```shell
root@deb-liferay:~# cat <<'EOF' > /opt/liferay/tomcat/bin/setenv.sh
"CATALINA_OPTS="$CATALINA_OPTS"
EOF
```

* Create Liferay daemon file

```shell
root@deb-liferay:~# nano /etc/systemd/system/liferay.service
```
> Insert bellow lines and configure as enough as yours system hardware.
```shell
[Unit]
Description=Liferay Apache Tomcat Web Application Container
After=network.target

[Service]
Type=forking
Environment=JAVA_HOME=/usr/lib/jvm/adoptopenjdk-8-hotspot-amd64
Environment=CATALINA_PID=/opt/liferay/tomcat/temp/tomcat.pid
Environment=CATALINA_HOME=/opt/liferay/tomcat
Environment=CATALINA_BASE=/opt/liferay/tomcat
Environment='CATALINA_OPTS= -Xms8g -Xmx8g -XX:MaxMetaspaceSize=1g -XX:MetaspaceSize=1g -XX:NewSize=4g -XX:MaxNewSize=4g -XX:SurvivorRatio=7 -server -XX:+UseParallelGC '
Environment='JAVA_OPTS= -Djava.awt.headless=true -Djava.security.egd=file:/dev/./urandom -Dfile.encoding=UTF-8 -Djava.locale.providers=JRE,COMPAT,CLDR -Djava.net.preferIPv4Stack=true -Duser.timezone=GMT '

ExecStart=/opt/liferay/tomcat/bin/startup.sh
ExecStop=/opt/liferay/tomcat/bin/shutdown.sh

User=liferay
Group=liferay
UMask=0007
RestartSec=10
Restart=always

[Install]
WantedBy=multi-user.target

```
> Add liferay user and grant permissions

```shell
root@deb-liferay:~# useradd -M -U -s /bin/false liferay
root@deb-liferay:~# chown -Rf liferay:liferay /opt/liferay*
```
> Enable and Start Liferay daemon

```shell
root@deb-liferay:~# systemctl enable liferay
Created symlink /etc/systemd/system/multi-user.target.wants/liferay.service → /etc/systemd/system/liferay.service.
root@deb-liferay:~# systemctl start  liferay
root@deb-liferay:~# root@deb-liferay:~# tail -f /opt/liferay/tomcat/logs/catalina.out
```

## Installing Nginx
We need to install `nginx` to active `ssl` and some caching purpose

```shell
root@deb-liferay:/etc/nginx/conf.d# apt install nginx
```
> configure nginx

create file in `/etc/nginx/conf.d/YOUR-DOMAIN.conf` and insert bellow line in this files

```shell
server {
    listen 80 ;
    server_name  SERVER-IP YOUR-DOMAIN;

location / {
            proxy_pass                  http://127.0.0.1:8080;
            root                        html;
            index                       index.html index.htm;
            proxy_redirect              off;
            proxy_set_header            Host $http_host;
            proxy_set_header            X-Forwarded-Server $host;
            proxy_set_header            X-Real-IP $remote_addr;
            proxy_set_header            X-Forwarded-For $proxy_add_x_forwarded_for;
            client_max_body_size        100m;
            client_body_buffer_size     1024k;
            proxy_connect_timeout       120;
            proxy_send_timeout          120;
            proxy_read_timeout          120;
            proxy_buffer_size           4k;
            proxy_buffers               4 32k;
            proxy_busy_buffers_size     64k;
            proxy_temp_file_write_size  64k;

          }
}
```
> **Note:** If you want to use server ip you must chane the `redirect.url.ips.allowed` property in `portal-ext.properties` like bellow and restart the liferay.

```shell
redirect.url.ips.allowed=127.0.0.1,SERVER-IP
```

## Securing
We install `firewalld` and open ports fro only `http,htpps,ssh` services.
first disable `ufw` if its exist.

```shell
sudo ufw disable
```

```shell
root@deb-liferay:~# sudo apt -y install firewalld
root@deb-liferay:~# firewall-cmd --add-service=http --permanent
```

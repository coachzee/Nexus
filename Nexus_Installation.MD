# Nexus Installation
Nexus is one a artifact repository which helps to store your build outcomes.  

Prerequisites
1. Make Sure You Have Java Installed (Refer To Java Raequirement On Official Website)
2. Validate Below Files 

Update the /etc/security/limits.conf file 

```sh
# vim /etc/security/limits.conf

root soft memlock unlimited
root hard memlock unlimited
root - nofile 65536
root - nproc 4096
nexus - nofile 65536
nexus - nproc 4096
```

Update file with below entry the /etc/sysctl.conf file and reload sysctl

```sh
# vim /etc/sysctl.conf

fs.file-max = 2097152
vm.max_map_count = 262144
vm.swappiness = 1

# sysctl -p /etc/sysctl.conf
```

# Implementation steps 
Download and setup nexus stable version

```sh 
cd /opt
```
```sh 
wget https://download.sonatype.com/nexus/3/latest-unix.tar.gz
```
Untar Nexus Latest Package
```sh 
tar -zxvf  latest-unix.tar.gz
```

Rename Untar File To "nexus"  in "/opt/nexus" 

```sh 
mv /opt/nexus-3.32.0-03 /opt/nexus
```

As a good security practice, it is not advised to run nexus service as root. so create new user called nexus and grant sudo access to manage nexus services 

Create nexus service account 
```sh 
useradd nexus
```
As root Change Ownership Of the two files extracted  to nexus  like below "nexus" and "sonatype-work":
```sh 
sudo chown -R nexus:nexus /opt/nexus
sudo chown -R nexus:nexus /opt/sonartype-work
```
Edit and uncomment below file to run as nexus

```sh 
vi /opt/nexus/bin/nexus.rc
```
```sh 
run_as_user="nexus" (file shold have only this line and dont forget to uncomment "remove #")
```
Add nexus as a service at boot time
```sh
sudo ln -s /opt/nexus/bin/nexus /etc/init.d/nexus
```
Login as a nexus user and start service

```sh
su - nexus
service nexus start
```
Validate VMoption Configuration 
```sh 
vim /opt/nexus/bin/nexus.vmoptions
```

```sh
-Xms2703m
-Xmx2703m
-XX:MaxDirectMemorySize=2703m
-XX:+UnlockDiagnosticVMOptions
-XX:+LogVMOutput
-XX:LogFile=../sonartype-work/nexus3/log/jvm.log
-XX:-OmitStackTraceInFastThrow
-Djava.net.preferIPv4Stack=true
-Dkaraf.home=.
-Dkaraf.base=.
-Dkaraf.etc=etc/karaf
-Djava.util.logging.config.file=etc/karaf/java.util.logging.properties
-Dkaraf.data=../nexusdata/nexus3
-Dkaraf.log=../sonartype-work/nexus3/log
-Djava.io.tmpdir=../sonartype-work/nexus3/tmp
-Dkaraf.startLocalConsole=false
```
Modify nexus-default.properties file.

```sh
vim /opt/nexus/etc/nexus-default.properties
```
Set Nexus as a System Service

```sh
vim /etc/systemd/system/nexus.service
``` 
 ```sh
 [Unit]
Description=Nexus Service
After=syslog.target network.target

[Service]
Type=forking
LimitNOFILE=65536
ExecStart=/opt/nexus/bin/nexus start
ExecStop=/opt/nexus/bin/nexus stop
User=nexus
Group=nexus
Restart=on-failure

[Install]
WantedBy=multi-user.target
```
Start, Reload Daemon $ Enable nexus service 
```sh
systemctl daemon-reload
systemctl start nexus.service
systemctl enable nexus.service
```
 
Login nexus server from browser on port 8081
```sh
http://<Nexus_server>:8081
```
The path to default password will be provided such as "/opt/sonatype-work/nexus3/admin.password"

```sh
username : admin  
password : "/opt/sonatype-work/nexus3/admin.password"
```
#Troubleshooting

#service is not starting?
- make sure you are trying to start service with nexus user. 
- check java installation

Unable to access nexus URL?
- make sure port 8081 is opened.

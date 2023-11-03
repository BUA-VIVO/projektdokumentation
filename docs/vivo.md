# System requirements

If time has passed between the server setup and the installation, apt should be updated again:

```sh
sudo apt update
```

**Open Java Developer Kit:**

```sh
sudo apt install default-jdk
```

**Maven**

```sh
sudo apt install maven
```

**Curl**

```sh
sudo apt install curl
```

**git**

```sh
sudo apt install git
```

**Apache Tomcat**

Tomcat does not have its own user group here, as VIVO lives and works in the Tomcat directory.

The Tomcat download can in principle take place anywhere, but it is advisable to execute it in the /tmp folder.

```sh
cd /tmp
curl -O http://www-eu.apache.org/dist/tomcat/tomcat-9/v9.0.11/bin/apache-tomcat-9.0.11.tar.gz
```

Tomcat is installed in the opt/tomcat directory:

```sh
sudo mkdir /opt/tomcat
sudo tar xzvf apache-tomcat-9*tar.gz -C /opt/tomcat --strip-components=1
```

Create a systemd service file for Tomcat:

```sh
sudo nano /etc/systemd/system/tomcat.service
```

with the following content:

```text
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

UMask=0007
RestartSec=10
Restart=always

[Install]
WantedBy=multi-user.target
```

Vivo recommends including `-XX:MaxPermSize=128m` in the CATALINA\_OPTS. VIVO also recommends a max heap of 512m, we are currently working with 1024M. So far there has been no loss of performance.

To activate UTF-8 compatibility in Tomcat, open the setup.xml:

```sh
nano /opt/tocat/conf/server.xml
```

and activate URI encoding in the connector elements:

```xml
 <Server ...>

  <Service ...>

    <Connector ... URIEncoding="UTF-8"/>

      ...

    </Connector>

  </Service>

</Server> 
```

Start / stop Tomcat

```sh
sudo systemctl start tomcat
sudo systemctl stop tomcat
```

# **VIVO 12**

**Download**

VIVO can be downloaded to a folder in /tmp or to the user directory. On the test server it is in the user directory for easy accessibility, on the live server it is loaded into /tmp.

```sh
git clone https://github.com/vivo-project/Vitro.git Vitro -b rel-1.12.2-maint
git clone https://github.com/vivo-project/VIVO.git VIVO -b rel-1.12.2-maint
git clone https://github.com/vivo-project/Vitro-languages.git Vitro-languages -b rel-1.12.2-maint 
git clone https://github.com/vivo-project/VIVO-languages.git VIVO-languages -b rel-1.12.2-maint 
```

The Vivo installation files are no longer needed after installation and can be deleted. On the test server, the files are stored in the home directory in case of a new installation.

**Installation**

```sh
mvn install -s example-settings.xml
```

The installation creates 2 folders:

*   vivo-home: /usr/local/vivo -&gt; contains the TripleStores and SPARQL queries for the page structure and Java code,...
*   webapps: /opt/tomcat/webapps/vivo -&gt; contains the templates, images, texts, front-end information, ...

Beide Ordner sind in GitHub gespiegelt:

The installation folder in the usr directory: [https://github.com/BUA-VIVO/vivo-home-config](https://github.com/BUA-VIVO/vivo-home-config)

The webapps folder in the tomcat directory: [https://github.com/BUA-VIVO/vivo-frontend](https://github.com/BUA-VIVO/vivo-frontend)

**Nach der Installation - Webapps**

The frontend can be pulled directly from GitHub after installation:

1.  delete the entire content of /opt/tomcat/webapps/vivo

	```sh
	sudo rm -r /opt/tomcat/webapps/vivo/*
	```

2.  clone the repository from GitHub into a folder of your choice

	```sh
	git clone https://github.com/BUA-VIVO/vivo-frontend main
	```
3. create a symbolic link from the repository in /opt/tomcat/webapps

	```sh
		ln -s /path/to/frontend-repository /opt/tomcat/webapps/desired_name_of_webapp
	```
	
If the webapp is to be ran as the root web application of Tomcat, rename the original ROOT folder and create the following link

```sh
	ln -s /path/to/frontend-repository /opt/tomcat/webapps/ROOT
```
	

A restart of the Tomcat server is not necessary in principle, but is recommended.

To display the frontend updates in the browser, the browser cache must be deleted.


**Nach der Installation - vivo-home**

Before using vivo-home for the first time, the runtime.properties must be adapted.


```text
Vitro.defaultNamespace = http://vivo.berlin-university-alliance.de/individual/

rootUser.emailAddress = xxx@yyy.zzz

vitro.local.solr.url = http://192.168.10.20:8983/solr/vivocore
```

The rootUser.emailAddress is only relevant for the first login and may differ.
The IP in the SolR URL must be adjusted. 

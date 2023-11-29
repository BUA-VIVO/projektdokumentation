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

## **VIVO 12**

### **Download**

VIVO can be downloaded to a folder in /tmp or to the user directory. On the test server it is in the user directory for easy accessibility, on the live server it is loaded into /tmp.

```sh
git clone https://github.com/vivo-project/Vitro.git Vitro -b rel-1.12.2-maint
git clone https://github.com/vivo-project/VIVO.git VIVO -b rel-1.12.2-maint
git clone https://github.com/vivo-project/Vitro-languages.git Vitro-languages -b rel-1.12.2-maint 
git clone https://github.com/vivo-project/VIVO-languages.git VIVO-languages -b rel-1.12.2-maint 
```

The Vivo installation files are no longer needed after installation and can be deleted. On the test server, the files are stored in the home directory in case of a new installation.

### **Installation**

```sh
mvn install -s example-settings.xml
```

The installation creates 2 folders:

*   vivo-home: /usr/local/vivo -&gt; contains the TripleStores and SPARQL queries for the page structure and Java code,...
*   webapps: /opt/tomcat/webapps/vivo -&gt; contains the templates, images, texts, front-end information, ...

Beide Ordner sind in GitHub gespiegelt:

The installation folder in the usr directory: [https://github.com/BUA-VIVO/vivo-home-config](https://github.com/BUA-VIVO/vivo-home-config)

The webapps folder in the tomcat directory: [https://github.com/BUA-VIVO/vivo-frontend](https://github.com/BUA-VIVO/vivo-frontend)

## **After Installation - Webapps**

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


### **BUA Vivo Webapp Custom alterations to the freemarker template files**

Vivo is a Java application, and as such it uses the [Freemarker Template Engine](https://freemarker.apache.org/index.html) to render the html of the web application.
The changes made to the templates mainly reside in the [vivo-frontend repository
](https://github.com/BUA-VIVO/vivo-frontend.git),specifically for the ***tenderfoot*** template,  but an important configuration regarding the way the frontend renders the front page aggregated lists is additionally found in the ['vivo home config' repository](https://github.com/BUA-VIVO/vivo-home-config), that is, in the h[ome page data getter configuration file](https://github.com/BUA-VIVO/vivo-home-config/blob/main/rdf/display/everytime/homePageDataGetters.n3) ***rdf/display/everytime/homePageDataGetters.n3***

This file contains callable functions which perform [SPARQL queries
](https://www.w3.org/TR/sparql11-query/) fetching data for Berlin University alliance members, the participating Excellence Clusters as well as for Projects in the graph.

The bua member function "***display:buaDataGetter***":

```sparql
	display:buaDataGetter
	    a <java:edu.cornell.mannlib.vitro.webapp.utils.dataGetter.SparqlQueryDataGetter> ;
	    display:saveToVar "buaMitglieder" ;
	    display:query """
	    PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
	    PREFIX vivo: <http://vivoweb.org/ontology/core#>
	    PREFIX vivo-de: <http://vivoweb.org/ontology/core/de#>
	    PREFIX vivo-bua: <http://vivo.berlin-university-alliance.de/ontology/core/v1/vivo/bua#>
	    PREFIX bua: <http://vivo.berlin-university-alliance.de/ontology/core/v1/bua/>
	    PREFIX obo: <http://purl.obolibrary.org/obo/>
	
	    SELECT DISTINCT ?uri ?label 
	    WHERE 
	    {
	       bua:bua obo:BFO_0000051 ?uri .
	       OPTIONAL { ?uri rdfs:label ?label . 
	       FILTER (lang(?label) = "de-de") } .
	     }
	     ORDER BY ?label
	     """ .
```

* Lines 2-10 are the namespaces needed to resolve the SPARQL query
* line 12 asks to select the uri identifier and the label of each entity
* line 15, asks to get the uri for any entity being playing the role of ***obo:BFO_0000051 (has part)*** for the ***bua*** entity
* line 16 -17 fetches the labels (names) for the entities in the german language perspective (de-de)

**The display:buaProjectDataGetter (fetch projects) getter function:**

```sparql
display:buaProjectDataGetter
	  a <java:edu.cornell.mannlib.vitro.webapp.utils.dataGetter.SparqlQueryDataGetter> ;
	  display:saveToVar "buaProjekte" ;
	  display:query """
	  PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
	  PREFIX vivo: <http://vivoweb.org/ontology/core#>
	  
	  SELECT DISTINCT ?uri ?label
	  WHERE
	    {
	       ?uri a vivo:Project .
	       OPTIONAL { ?uri rdfs:label ?label} . 
	     }
	     ORDER BY ?label
	  """ .
```
* Lines 2-6 are the namespaces needed to resolve the SPARQL query
* line 8 asks to select the uri identifier and the label of each entity
* line 11, asks to get the uri for any entity instantiating the ***vivo:Project*** class
* line 12 fetches the labels (names) for the projects


**The display:exzellenzClustersDataGetter (fetch excellence clusters) getter function:**

```sparql
display:exzellenzClustersDataGetter
	a <java:edu.cornell.mannlib.vitro.webapp.utils.dataGetter.SparqlQueryDataGetter> ;
    display:saveToVar "exzellenzClusters" ;
    display:query """
    PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
    PREFIX vivo: <http://vivoweb.org/ontology/core#>
    PREFIX vivo-de: <http://vivoweb.org/ontology/core/de#>
    PREFIX bua: <http://vivo.berlin-university-alliance.de/ontology/core/v1/bua/>
    
    SELECT DISTINCT ?uri ?label 
    WHERE 
    {
       ?uri a bua:Exzellenzcluster .
       OPTIONAL { ?uri rdfs:label ?label . 
       FILTER (lang(?label) = "en-us") } .
     }
     ORDER BY ?label
     """ .
```

* Lines 2-8 are the namespaces needed to resolve the SPARQL query
* line 10 asks to select the uri identifier and the label of each entity
* line 13, asks to get the uri for any entity instantiating the  ***bua:Exzellenzcluster*** class
* line 14 -15 fetches the labels (names) for the entities in the english language perspective (de-de)

####Calls to the functions in the web app template code####

#####**templates/freemarker/lib/lib-home-page.ftl:**#####

Renders the html for the faculty member section on the home page, Works in conjunction with the homePageUtils.js file, which contains the ajax call
The data is parsed from the variables created in the SPARQL queries of the **homePageDataGetters.n3** file and made ready for calls from **themes/tenderfoot/templates/page/page-home.ftl**

#####**js/homePageUtils.js**#####
contains the functions

* buildExzellenzClusters()
* buildBuaMitglieder()
* buildBuaProjects()

which build the html for rendering, reading the corresponding variables in the **lib-home-page.ftl**

#####**themes/tenderfoot/templates/page/page-home.ftl**#####

Calls and renders the data generated in **lib-home-page.ftl**

```html
<div class="container">
    <div class="col-md-4">
        <!-- List List of bua members -->
        <@lh.buaMitGliederHtml />
    </div>
    <div class="col-md-4">
        <!-- List of exzellenzclusters -->
        <@lh.exzellenzClusterHtml />
    </div>
    <div class="col-md-4">
        <!-- List of projects -->
        <@lh.buaProjectHtml />
    </div>
</div>
```

#### Language files ####
The templates read the strings for the labels in the web application from i18n language configuration files found in the  ***i18n*** folder,
 specifically, the ***vivo_all_de_DE.properties*** and the ***all_de_DE.properties*** files are altered
 
 


## **After the Installation - vivo-home**

Before using vivo-home for the first time, the runtime.properties must be adapted.


```text
Vitro.defaultNamespace = http://vivo.berlin-university-alliance.de/individual/

rootUser.emailAddress = xxx@yyy.zzz

vitro.local.solr.url = http://192.168.10.20:8983/solr/vivocore
```

The rootUser.emailAddress is only relevant for the first login and may differ.

The IP in the SolR URL must be adjusted. 

In addition, the location of the home folder has to be configured in the webapp in frontend-repository

Open **vivo-frontend/META-INF/context.xml** and
change the path of the Environment/value attribute:

```xml
<Context>
   <Environment
           type="java.lang.String"
           name="vitro/home"
           value="....PATH TO ...../vivo-home-config" override="true"/>
           
   <Manager pathname="" />
</Context>
```
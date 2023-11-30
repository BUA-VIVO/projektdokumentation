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
The templates read the strings for the labels in the web application from i18n language configuration files found in the  i18n folder, specifically, the 

* [vivo\_all\_de\_DE.properties](https://raw.githubusercontent.com/BUA-VIVO/vivo-frontend/main/i18n/vivo_all_de_DE.properties)
* [all\_de\_DE.properties](https://raw.githubusercontent.com/BUA-VIVO/vivo-frontend/main/i18n/all_de_DE.properties)

and 

* [themes/tenderfoot/i18n/all\_en\_US.properties](https://raw.githubusercontent.com/BUA-VIVO/vivo-frontend/main/themes/tenderfoot/i18n/all_en_US.properties)
 

####CSS ALTERATIONS####
For the BUA VIVO project, some alterations and additions to some of the tenderfoot theme stylesheets where implemented:

#####**[themes/tenderfoot/css/page-home.css](https://raw.githubusercontent.com/BUA-VIVO/vivo-frontend/main/themes/tenderfoot/css/page-home.css)**#####
```css

.jumbotron {
    background-color: transparent;
    color: #ffffff;
    text-align: center;
    text-shadow: 1px 1px 4px #153f6c;
    
}

.jumbotron p {
    color: #fff;
	text-shadow: 1px 1px 15px #153f6c, 1px 1px 5px #153f6c, 1px 	1px 5px #153f6c;
	margin-top: 0px;
	bottom-top: 10px;
}


.jumbotron h1 {
	font-size: 32px;
	margin-top: 20px;
} /*added by vivo-bua to change intro font size*/

.hero {
    /*background: #283a4b url(../images/hero-backgroundBUA.png) repeat-x 0 0;*/
	background: linear-gradient(rgba(7,65,47, 0), rgba(7,65,47, 0)), url(../images/hero-backgroundBUA.png);
  height: 100%;

  /* Position and center the image to scale nicely on all screens */
  background-position: center;
  background-repeat: no-repeat;
  background-size: cover;
  position: relative;

}

.home-sections {
    border-top: 1px dotted #dbe3e3; /* stroke */
    border-bottom: 0px dotted #dbe3e3; /* stroke */
    background-color: #fff; /* layer fill content */
}

.home-sections h4 {
    border-top: 1px solid rgba(220,228,227,.42); /* stroke */
    border-bottom: 1px solid rgba(220,228,227,.42); /* stroke */
    /* background-color: #395d7f; /* layer fill content */
    background-color: #4DB178;
    border-bottom: 2px solid #86C8A3;
    color: #fff;
}

div.faculty-home {
    padding-top: 60px;
}

section#home-bua-mitglieder > div#bua-mitglieder > ul {
  margin: 0;
  border: 0;
  padding-left: 40px;
  padding-top: 10px;
}

section#home-exzellenz-cluster > div#exzellenz-Clusters > ul {
  margin: 0;
  border: 0;
  padding-left: 40px;
  padding-top: 10px;
}

section#home-bua-project > div#bua-projects > ul {
  margin: 0;
  border: 0;
  padding-left: 40px;
  padding-top: 10px;
}

section#home-bua-project > div#bua-projects > ul > li {
  padding-bottom: 5px;
}

section#home-research > ul {
  margin: 0;
  border: 0;
  padding-left: 30px;
  padding-top: 10px;
}


```
#####**themes/tenderfoot/css/tenderfoot.css**#####
```css

.navbar-toggle .button-label {
    display: inline-block;
    float: left;
    font-weight: bold;
    color: #4EB178; 
    line-height: 14px;
    padding-right: 10px;
}

body {
    padding: 0;
    height: 100%; /* needed for container min-height */
    font-family: "Noto Sans", "Lucida Sans Unicode","Lucida Grande", Geneva, helvetica, sans-serif;
    height: auto !important; /* real browsers */
    height: 100%; /* IE6: treaded as min-height*/;
    min-height: 100%; /* real browsers */
    margin: 0 auto;
   /* background: #f3f3f0 url(../images/header-backgroundBUA.png) center 0 no-repeat;*/

  /*  background: linear-gradient(rgba(7,65,47, 1),rgba(7,65,47, 1), rgba(199,205,205,1)), url(../images/header-backgroundBUA.png);*/

   /*  background: linear-gradient(180deg, #07412F, #07412F, #C7CDCD, #C7CDCD, #C7CDCD); */

/* background: linear-gradient(180deg, #360860,#360860,#E2DEE6, #E2DEE6,#E2DEE6);*/

background: linear-gradient(rgba(7,65,47, 1),rgba(7,65,47, 1), rgba(7,65,47, 0.5), rgba(211,215,215,1),rgba(211,215,215,1));

  /* Position and center the image to scale nicely on all screens */
  background-position: center;
  background-repeat: no-repeat;
  background-size: cover;
  position: relative;

}


#wrapper-content {
    background: repeat scroll 0 0 #fff;
    padding-top: 20px;
    padding-bottom: 20px;
}

.row.title {
    background: #07412F;
    padding-top: 7px;
    padding-bottom: 3px;
}

.person-details {
    background: #f3f3f;
    padding-bottom: 5px;
}


#search-field {
    width: 396px;
    height: 38px;
    background: url(../images/search-interior-pages.png) 0 0 	no-repeat;
	background-size: cover;
}


.icon-search {
    text-decoration: none;
    background-color: transparent;
    color: #4DB178;
    font-size: 14px;
    border: none;
    cursor: pointer;
}

footer .row {
	background-color: #07412F;
	/*  background-color: black;
	background-color: rgba(0, 0, 0, 0);*/
}


footer p.partners {
    text-align: center;
    margin-top: 20px;
    color: white;
	font-size: 14px;
}

ul#footer-nav {
    float: right;
    list-style: none;
    height: 20px;
    margin: 0;
    padding: 0;
    padding-top: 15px;
}

#footer-nav a:hover,
a.terms, a.partners,
a.powered-by-vivo {
    color: white;
    text-decoration: none;
}


#footer-nav  a.terms:hover, a.partners,
a.powered-by-vivo:hover {
    color: white;
    text-decoration: none;
}

/* BRANDING ------>  BUA LOGO */
h1.vivo-logo {
    position: absolute;
    width: 100%;
    max-width: 442px;
    height: 59px;
    top: 30px;
    left: 0;
    /* background: url(../images/VIVO-logo.png) 0 0 no-repeat; */
    background: url(../images/BUA_logo.svg) 0 0 no-repeat;
    background-size: 100% auto;
}


```


#### BUA IMAGES ####
Some images specific to the BUA VIVO were created:

* [themes/tenderfoot/images/BUA_logo.png](https://github.com/BUA-VIVO/vivo-frontend/blob/main/themes/tenderfoot/images/BUA_logo.png)
* [themes/tenderfoot/images/BUA_logo.svg](https://github.com/BUA-VIVO/vivo-frontend/blob/main/themes/tenderfoot/images/BUA_logo.svg)
* [themes/tenderfoot/images/Footer.png](https://github.com/BUA-VIVO/vivo-frontend/blob/main/themes/tenderfoot/images/Footer.png)
* [themes/tenderfoot/images/header-backgroundBUA.png](https://github.com/BUA-VIVO/vivo-frontend/blob/main/themes/tenderfoot/images/header-backgroundBUA.png)
* [themes/tenderfoot/images/hero-backgroundBUA.png
](https://github.com/BUA-VIVO/vivo-frontend/blob/main/themes/tenderfoot/images/hero-backgroundBUA.png)


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
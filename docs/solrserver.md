# SolR Server

## Setup

If time has passed between the server setup and the installation, apt should be updated again:

```sh
sudo apt update
```

### Open Java Developer Kit:

```sh
sudo apt install default-jdk
```

### git

```sh
sudo apt install git
```

### Downloading SolR (8.11.2)

By now version 9.4.0 is available and should be preferred

```sh
wget https://www.apache.org/dyn/closer.lua/lucene/solr/8.11.2/solr-8.11.2-src.tgz?action=download
```

### Downloading vivocore

```sh
git clone https://github.com/vivo-project/vivo-solr.git
```

### Package Installation

Install to local home directory.

```sh
cd ~/
tar zxf solr-8.11.2.tgz
```

### Add vivocore to SolR

```sh
cd ~/
cp -r vivo-solr/vivocore solr-8.11.2/server/solr/
```

### Start SolR

```sh
cd ~/solr-8.11.2
/bin/solr start
```

### Delete schema.xml

schema.xml was created on startup and should be deleted

```sh
cd ~/
rm solr-8.11.2/server/solr/vivocore/conf/schema.xml
```

## Follow Ups

### Update VIVO runtime.properties

The solr source in the VIVO runtime properties needs to point to the SolR URL

```xml
vitro.local.solr.url = http://192.168.xxx.xxx/solr/vivocore
```

### Restart VIVO

If Vivo was running before the setup of the SolR Server, it needs to be restarted

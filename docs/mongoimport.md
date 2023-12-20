# Import into MongoDB

The information extracted by the [Cluster Scripts](https://bua-vivo.github.io/projektdokumentation/clusters/) are transferred into a MongoDB Database on the [MongoDB-Server](https://bua-vivo.github.io/projektdokumentation/mongoserver/). A python [script](https://github.com/BUA-VIVO/bua-vivo-pipelines/blob/main/VivoPackage/Input2Mongo.py) is used to facilitate the transfer.  A second [script](https://github.com/BUA-VIVO/bua-vivo-pipelines/blob/main/VivoPackage/mongo/dbSchema.py) defines the MongoDB-Classes.

## Setting up a Database

The reccomended way to set up a database is the using MongoDB Compass App.
The sidebar button `new Connection` lets connect to the MongoDB-Server using a URI.  
The  string has the follwing format:
```
mongodb://USERNAME:PASSWORD@ServerIP:PORT/
```
The Mongo Standard Port is 27017.

After a connection is establishes the sidebar offers shortlinks for `My Queries` and `Databases`. Behind the Databases-Link the `+`-Symbol creates a new Database.   
Databases are deleted if all Collections inside are deleted. It is advised to create an empty Admin-Collection inside the every new Database, to avoid the deletion of the Database in case of a reset.   

## Prerequisites

### Mongoengine
To establish connections between Python and MongoDB [mongoengine](https://pypi.org/project/mongoengine/) is used. To install use:
```
pip install mongoengine
```
For further documentation see the projects [read the docs](https://mongoengine-odm.readthedocs.io/apireference.html).

### DB Schema
In mongoengine, MongoDB Collections are defined using DB Schemas represented by Classes. Since MongoDB itself does not enforce schemata, and since schemata should be open to future additions, minimal schemata were used.  
Every Schema is defined in a Class which inherits from the DynamicDocument-Class. This allows to expand entries dynamically and retroactively. This offers the option of expanding on information stored as new data points become available.

The database is designed around the Vivo frontend. Therefore every entry in every database needs to have a name in order to be displayed in Vivo. Other data points are optional, even though in most cases a set of standard data points will form organically.  
The example shows the `Cluster`-Class. All other classes function comparably.  
```py
class Cluster(DynamicDocument):
    name = StringField(required=True)
```

The Classes are defined in the [dbSchema.py](https://github.com/BUA-VIVO/bua-vivo-pipelines/blob/main/VivoPackage/mongo/dbSchema.py).

## Pipeline

### Main

The [Input2Mongo.py](https://github.com/BUA-VIVO/bua-vivo-pipelines/blob/main/VivoPackage/Input2Mongo.py) functions as main handler for the pipeline.

It loads the Classes defined in the [Cluster Scripts](https://bua-vivo.github.io/projektdokumentation/clusters/) and instantiates them. An Import-List can be defined for the case, that only certain Clusters are to be imported.

```py
# load Cluster
moa = MattersOfActivity()
nc = NeuroCure()
sci = SCIoI()

# Define Clusters to import into MongoDB
importList = [moa,nc,sci]
```

Afterwards two additional Scripts are excecuted:
- [mongoPipelineForEntities](https://github.com/BUA-VIVO/bua-vivo-pipelines/blob/main/VivoPackage/mongo/mongoPipelineForEntities.py): writes singular entities (Cluster, Person,Project,Publication,Journal, Year) into the MongoDB
- [mongoPipelineForRelations](https://github.com/BUA-VIVO/bua-vivo-pipelines/blob/main/VivoPackage/mongo/mongoPipelineForRelations.py): writes relational data and connects the singular entities in the mongodb

### Entities

To transfer the entities into the MongoDB, a connection between the Script and the Database has to be established.

This is done using the `mongoengine`-Library:  
```py
from mongoengine import *
```
The connection is established with the same URI used under "Setting up a Database". Before the connection is established, `disconnect()` is used to make sure no other connection is stil active.

```py
MONGOURI: "mongodb://USERNAME:PASSWORD@ServerIP:PORT/"
disconnect()
connect(host=MONGOURI)
```

The clusterList is the list of Clusters defined in the "Main" section above. The methodList is a list of strings correlating to the Methods defined under [Classes and Methods](https://bua-vivo.github.io/projektdokumentation/clusters/#classes-and-methods) in the documentation of the Cluster Scripts.

```py
clusterList = importlist
methodList = ['Person','Cluster','Project','Publication','Journal']
```

The pipeline iterates over every Method per Cluster.
A try-except is used to make sure the used Method is defined in the respective Cluster Script. If a Method is not found a print statement informs about the Error and the Pipeline continues with the next Method.
If the Method is found, it is excecuted and the result is assigned to the variable `entityList`.

```py
try:
     meth = getattr(cluster,method)
 except:
     print(cluster.__name__(),method,'not found')
 else:
     entityList = meth()
```

Afterwards every single entity is preccessed according to the method that produced it. TQDM is simply used for visual status representation and can be omitted.
```py
for entity in tqdm(entityList):
  if method == 'Person':
    ...
  elif method == 'Cluster':
    ...
```

**Person:**  
First, middle and last name of an entity are read from the person dictionary.
```py
firstName = entity['firstName']
middleName = entity['middleName']
lastName = entity['lastName']
```
Then all objects are loaded from the MongoDB, where first and last name match the entity data.
```py
db = Person.objects(__raw__={'$and':[
                            {"firstName":{"$in":[firstName]}},
                            {"lastName":{"$in":[lastName]}}
                                ]})
```

If a matching dataset exists in the MongoDB, its information is updated to represent, that it is part of the current cluster.

```py
if db:
  updateOwnership(db,cluster)
```

The updateOwnership Function is incorporated in the [tools.py](https://github.com/BUA-VIVO/bua-vivo-pipelines/blob/main/VivoPackage/mongo/tools.py). It opens the corresponding database entry, checks if the cluster is already part of the flag_owner entry and adds it, if that is not already the case.

```py
def updateOwnership(db,cluster):
    obj = db[0]
    owner = obj.flag_owner
    if cluster.__name__() not in owner:
        owner.append(cluster.__name__())
        obj.flag_owner = owner
        obj.save()
```

In case no matching Dataset is found in the MongoDB, a new entry is created and saved. While creating the entry, a conditional for middle name inclusion is executed when the full name is created. The flag for the owning cluster is also saved as part of the entry.

```py
if middleName == '':
    name = firstName+' '+lastName
else:
    name = firstName+' '+middleName+' '+lastName
Person(name= name,firstName = firstName, middleName=middleName,lastName=lastName,flag_owner=[cluster.__name__()]).save()
```

**Cluster, Projects and Journals:**  
The Cluster-, Projects- and Journal-pipelines works comparibly. They check for errors in the name variable before loading the respective dataset with the corresponding name. If this dataset exists, it is updated using the update 'updateOwnership'-Function otherwise it is written and saved to the MongoDB.

```py
name = entity['name']
if name not in ['','None']:
    db = Cluster.objects(name=name)
    if db:
        updateOwnership(db,cluster)
    else:
        db = Cluster(name=name,flag_owner=[cluster.__name__()]).save()
```

**Publications:**  

Publications use the same template. When loading entries from the Database it is not only checked for a matching name but also for a matching publication year. This is done to avoid overwriting Publications with similar names.

```py
db = Publication.objects(__raw__={'$and':[
                                  {"name":{"$in":[name]}},
                                  {"year":{"$in":[year]}}
                                      ]})
```

**Years:**  

Years are created independently of the cluster loop. For dating purposes inside Vivo, as of right now only years are necessary. An entry for every year between 1500 and 2200 is created. If en error is detected in the Database (e.g. a year in this range is missing) the complete Year-Collection will be newly created and overwritten.

```py
yearList = Year.objects()
if not all(item in [x.name for x in yearList] for item in list(range(1500,2200))):
    for year in tqdm(range(1500,2200)):
        yearObject = Year(name = str(year))
        yearObject['@type'] = 'http://vivoweb.org/ontology/core#DateTimeValue'
        yearObject['dateTime'] = str(year)+'-01-01T12:00:00'
        yearObject['dateTimePrecision'] = 'http://vivoweb.org/ontology/core#yearPrecision'
        yearObject.save()
```

### Relations

To transfer the entities into the MongoDB, a connection between the Script and the Database has to be established.

This is done using the `mongoengine`-Library:  
```py
from mongoengine import *
```
The connection is established with the same URI used under "Setting up a Database". Before the connection is established, `disconnect()` is used to make sure no other connection is stil active.

```py
MONGOURI: "mongodb://USERNAME:PASSWORD@ServerIP:PORT/"
disconnect()
connect(host=MONGOURI)
```

The clusterList is the list of Clusters defined in the "Main" section above. The methodList is a list of strings correlating to the Methods defined under [Classes and Methods](https://bua-vivo.github.io/projektdokumentation/clusters/#classes-and-methods) in the documentation of the Cluster Scripts.

```py
clusterList = importlist
methodList = ['Person','Cluster','Project','Publication','Journal']
```

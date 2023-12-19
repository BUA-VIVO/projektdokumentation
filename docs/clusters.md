# Clusters

The following scripts are used to standardize data for further usage in the pipelines.

## Input

Since data for the project is accepted from different sources and in different formats, data cleaning steps are required. The associated Clusters of Excellence "Matters of Activity", "NeuroCure" and "Science of Intelligence" handed in data in `xlsx` and `json` format. NeuroCure allowed the usage of publicly available data on [ORCiD](https://orcid.org/).

## Output

### Output Data

For every cluster data was collected on:  
- Cluster Information  
- Persons
- Projects
- Research Output
- Journals

### Output formats

The following output formats define the  minimum amount of information required for a working database in order to link information together properly. More information can be and is often provided. Additional data will be as flat key:value pairs and should be provided accordingly.

**Cluster Information:**  
For Clusters, a dictionary is returned.
```json
 [{"name": "CLUSTERNAME",
  "overview": "CLUSTER DESCRIPTION",
  "uri": "URI*"}]
```

\* URI must match entries in the [BUA VIVO Ontology Extrension](https://github.com/BUA-VIVO/bua-vivo-ontology-extensions/blob/main/vivo-bua-ext.rdf)

**Persons:**  
Persons can either be Cluster Members or it can be associated co-workers like Editors, co-authors, etc. For data protection purposes, only minimal biographical data of associates is processed. The output format is a list of dictionaries per Cluster.

 ```json
[{"firstName":"FIRSTNAME",
  "middleName": "MIDDLENAME",
  "lastName":"LASTNAME",
  "givenName":"GIVENNAME",
  "familyName":"FAMILYNAME",
  "label":"LABEL*",
  "participatesIn":"CLUSTER-URI**"
  },
  {...}
]
 ```

\* LABEL is a human readable form of first, middle and last Name to be displayed on the Vivo plattform

\** CLUSTER-URI must match entries in the [BUA VIVO Ontology Extrension](https://github.com/BUA-VIVO/bua-vivo-ontology-extensions/blob/main/vivo-bua-ext.rdf)

**Projects:**  
 Outputs a list of all Projects within a cluster. Associated members are linked to a project and  their respective roles are recorded.

 ```json
[{"name":"Project Name",
  "organisation": "CLUSTERNAME",
  "members":[{
        "firstName": "FIRSTNAME",
        "middleName":"MIDDLENAME",
        "lastName": "LASTNAME",
        "role":["ROLE_1","ROLE_2"]
        },
        {...}
        ],
  },
  {...}  
]
 ```

 **Research Output:**  
Outputs a list of all Research Output published within a cluster.

 ```json
 [{"name":"TITLE",
   "year": "YEAR",
   "authors":[{
         "firstName": "FIRSTNAME",
         "middleName":"MIDDLENAME",
         "lastName": "LASTNAME"
         },
         {...}
         ],
   },
   {...}  
 ]
 ```

 \* TYPE-URI must match ontology classes within the VIVO-Ontology or within the [BUA VIVO Ontology Extrension](https://github.com/BUA-VIVO/bua-vivo-ontology-extensions/blob/main/vivo-bua-ext.rdf)

 **Journals:**  
Returns a list of Journals, which served as publication venues for the cluster.

```json
[{"name": "JOURNALTITLE"},
 {...}
]
```

## Scripts

There needs to be one cleaning script per active data source. The scripts are written in Python and are used in the [Mongo Import Script](https://bua-vivo.github.io/projektdokumentation/mongoimport/). Momentarily there are 3 Scripts for each of the participating cluster:

- [Matters of Activity](https://github.com/BUA-VIVO/bua-vivo-pipelines/blob/main/VivoPackage/ClusterScripts/moaSetup.py)
- [NeuroCure](https://github.com/BUA-VIVO/bua-vivo-pipelines/blob/main/VivoPackage/ClusterScripts/NeuroCure.py)
- [Science of Intelligence](https://github.com/BUA-VIVO/bua-vivo-pipelines/blob/main/VivoPackage/ClusterScripts/SCIOI.py)


### Classes and Methods

Each Script consists of one Class (MattersOfActivity, NeuroCure and SCIoI).

The minimal `__init__`Method defines the Cluster-URI and the data sources used to obtain the data points. To oblidge to data protection regulation data sources in the code can not be provided and have to be manually adapted.

Matters of Activity is used as an example
for its minimalistic methods. If other scripts are cited, this will be explicitly mentioned-

#### Dunder

```python3
def __init__(self):
        self.publicationXLSX = 'MOA_PUBLICATION.xlsx'
        self.projectsJSON = 'MOA_PROJECTS.json'


        df_raw = pd.read_excel('./input/'+self.publicationXLSX)
        self.df = df_raw.fillna('None').reset_index(drop=True)
        self.uri ='http://vivo.berlin-university-alliance.de/ontology/core/v1/bua/matters-activity'
```

The `__init__`Methods of NeuroCure and SCIoI function similar, even though they handle more imports and/or further information like language or source.

Every Class has a defined name accessible using the `__name__`Method.

#### Cluster

```python3
def Cluster(self):
        overviewObj = '''
            The Cluster Matters of Activity.
            Image Space Material aims to create a basis for a new culture of materials.  [...]'''

        clusterDict = {}
        clusterDict.setdefault('name','Matters of Activity')
        clusterDict.setdefault('overview',overviewObj)
        clusterDict.setdefault('uri',self.uri)

        return [clusterDict]
```

The Cluster-Method defines a simple 3 entry dictionary. The Project name and description are hardcoded for every cluster, the URI is already defined in the `__init__`

#### Person

The source for the Person-Information varies by cluster. Matters of Activity (MoA) and SCIoI take it out of the pandas.DataFrame() defined in the `__init__`, for Neurocure JSON files were created with data from ORCiD API calls.

The amount of information provided and where to find specific it inside the datasets also varies by source.
MoA provides Names of affiliated people in 3 different columns of a excel sheet (often more than one per cell), while the NeuroCure data is already presorted.

```python3
ace = list(self.df['Authors/Contributors/Editors'])
eds = list(self.df['Editors'])
cm = list(self.df['Cluster members'])
```

The NeuroCure Script features two seperate functions. The first one is used to clean and split names retrieved from ORCiD.

```python3
def getnames(self, name):
        lastname_prefixes = ["le", "von", "van", "de"]
        names = {"firstName": "", "middleName": "", "lastName": ""}

        ns = name.split(" ")

        if len(ns) > 2:
            prefix_res = any(string in name.lower() for string in lastname_prefixes)
            if prefix_res:
                names['firstName'] = ns[0].strip().strip(",").strip(";").strip()
                names['middleName'] = " ".join(ns[1:-2]).strip().strip(",").strip(";").strip()
                names['lastName'] = " ".join(ns[-2:]).strip().strip(",").strip(";").strip()
            else:
                names['firstName'] = ns[0].strip().strip(",").strip(";").strip()
                names['middleName'] = " ".join(ns[1:-2]).strip().strip(",").strip(";").strip()
                names['lastName'] = ns[-1].strip().strip(",").strip(";").strip()

        else:
            names['firstName'] = ns[0].strip().strip(",").strip(";").strip()
            names['middleName'] = ""
            names['lastName'] = ns[-1].strip().strip(",").strip(";").strip()

        return names
```

 The second one is used to check if a proper name is provided in the ORDiC data and if this name is not a duplicate. Its return type is boolean.

 ```python3
 def hasname(self, val, foundnames):
        found = False
        if 'FirstName' in val and 'LastName' in val and len(val['LastName']) > 0 and len(val['FirstName']) > 0:
            if val['LastName'] + val['FirstName'] in foundnames or val['LastName'] + val['FirstName'][0] in foundnames or val['FirstName'] + val['LastName'][0] in foundnames:
                found = True
        elif 'firstname' in val and 'lastname' in val is not None and len(val['firstname']) > 0 and len(val['lastname']) > 0:
            if val['lastname'] + val['firstname'] in foundnames or val['firstname'] + val['lastname'] in foundnames:
                found = True
        return found
 ```

 They are then applied onto the ORCiD data of associated persons and onto the list of contributors of cluster publications.

 ```python3
 for _, val in enumerate(self.persons_info):
             if val not in foundelements and not self.hasname(val, foundnames):

  names = self.getnames(val['FirstName'] + ' ' + val['LastName'])
 ```

 After cleaning the data, all Person-Methods will then create a dictionary of lists containing all Person-data.

 ```python3
nameDict = {}
nameDict.setdefault('firstName',firstName)
nameDict.setdefault('middleName',middleName)
nameDict.setdefault('lastName',lastName)
nameDict.setdefault('givenName',givenName)
nameDict.setdefault('familyName',lastName)
nameDict.setdefault('label',label)
rawoutput.append(nameDict)
```

The MoA- and SCIoI-Scripts delete duplicates before returning the data. Neurocure skipps this step since it already deleted duplicates using the `hasname`-Method.

```python3
[output.append(x) for x in rawoutput if x not in output]

return output
```

#### Project

Equivalently to the Person Method, data is loaded from an external source, cleaned and returned as a list of dictionaries.

 In case of MoA, a list with all project names is created and ridded of all duplicates.

 ```python3
 projects = list(self.df['Achievement within the following projects'])
 for projectLine in projects:
     projectLineList = projectLine.split('//')
     for project in projectLineList:
         project = project.strip()
         if project != 'None':
             if project != 'No project context':
                 projectList.append(project)

 projectList = list(set(projectList))
 ```

 Afterwards every member associated to each project is extracted and saved within the project context.

 ```python3
 for row in range(len(self.df)):
   if pro in self.df['Achievement within the following projects'][row]:
       persons = self.df['Cluster members'][row]
       multiNameList = persons.split('//')
       for nameString in multiNameList:
           nameString = nameString.strip()
           nameList = nameString.split(' ')
           if len(nameList) == 2:
               firstName, lastName = nameList
               middleName = ''
           elif len(nameList) == 3:
               if '.' in nameList[1]:
                   firstName, middleName, lastName = nameList
               else:
                   firstName = nameList[0]
                   middleName = ''
                   lastName = nameList[1]+' '+nameList[2]
           else:
               firstName = nameList[0]+' '+nameList[1]
               middleName = ''
               lastName = nameList[2]+' '+nameList[3]
           personDict= {'firstName':firstName,
                        'middleName':middleName,
                        'lastName':lastName,
                        'role': ['member'] }
 ```

 Finally project information are collected inside dictionaries and appended onto a list containing projects belonging to a cluster.

 ```python3
 output = []
 projectDict = {}
 projectDict.setdefault('name',pro)
 projectDict.setdefault('organization','Matters of Activity')
 projectDict.setdefault('members',[])
 projectDict['members'].append(personDict)
 output.append(projectDict)
 return output
 ```

 #### Publication

 Publication Types are often specific to a certain Cluster. Often new data types must be integrated into the ontologies, before a a dataset can be uploaded. Therefore typedicts (hardcoded  dictionaries of all publication types a cluster has published and their respective URIs)  must be manually integrated into every Publication-Method.

 ```python3
 typedict = {
             'Edited Volume/Exhibition Catalogue': 'http://vivoweb.org/ontology/core/de/bua#EditedVolume',
             'Contribution in Edited Volume/Exhibition Catalogue': 'http://vivoweb.org/ontology/core/de/bua#ContributionInEditedVolume'
             }
 ```

A dictionary for every publication is created and filled with available information. The code snipped below shows how optional data can be handled and how conditional information can be integrated.

```python3
for row in range(len(self.df)):
    pubDict = {}
    pubDict.setdefault('name',self.df['Title of publication'][row])
    pubDict.setdefault('year', self.df['Year of publication'][row])
    if self.df['Abstract'][row] != 'None':
        pubDict.setdefault('abstract', self.df['Abstract'][row])
    if self.df['Volume'][row] != 'None':
        pubDict.setdefault('volume', self.df['Volume'][row])
    if self.df['Digital Object Identifier (DOI)'][row] != 'None':
        pubDict.setdefault('doi', self.df['Digital Object Identifier (DOI)'][row])
    if self.df['Title of journal/archive/magazine'][row] != 'None':
        pubDict.setdefault('hasPublicationVenue',self.df['Title of journal/archive/magazine'][row])
    if self.df['Type of publication'][row] != 'None':
        pubDict.setdefault('@type',typedict[self.df['Type of publication'][row]])

    pages = self.df['Pages (from - to)'][row]
    pageList = pages.split('-')
    if len(pageList) == 1:
        if pageList[0] not in ['None','np','Forthcoming']:
            pubDict.setdefault('startPage',pageList[0])
    elif len(pageList) == 2:
        pubDict.setdefault('startPage',pageList[0])
        pubDict.setdefault('endPage',pageList[1])
    else:
        pass
```

Name-lists are created as shown in the Person-Method and added to the dictionary. Dictionaries are appended to one list holding all publishing information of a cluster. That list will then be returned.

#### Journal

In case of MoA and SCIoI, Journals are collected as a list of Journal titles, extracted directly from one column of a excel sheet.

```python3
output = []
journalList = list(set(list(self.df_publ['Publication Title'])))
[output.append({'name': x}) for x in journalList]

return output
```

In case of NeuroCure, ORCiD data often provides years, which are recorded if available.

```python3
journal = {'name': val['journal-title'], 'year': val['publication-year']}
```

# Ontologies
Ontologies, are, in the context of Informatics, defined as "formal representation of concepts, their attributes and relations between them within a specific domain"

In our case, the project needed to work with Ontologies, in aiming to build a platform for researching, collecting and presenting scientists and their research as well as their activities within and across the members of the Berlin University Alliance (BUA), using the Linked Open Data software VIVO

[VIVO](https://github.com/vivo-project/VIVO) and more specifically [VITRO](https://github.com/vivo-project/Vitro) is software based on- and using Ontologies, originally representing and describing the American Academic Domain. As the entities and their terms found in the American academic domain in many cases diverge and also are ambigous in respect to the entities and their naming in the German (also European) academic domain, a greater part of the work within the BUA-project has been to create a mapping of the German domain.

The challenges found in creating an ontology for the German domain on basis of the original VIVO ontology is described in the bachelor thesis: "[Mapping und Erweiterung der Ontologie des Forschungsinformationssystems VIVO](https://doi.org/10.25968/opus-803)", and parallely, some work has already been done by the [VIVO-DE group](https://github.com/VIVO-DE/vivo-de-ontology-extension). 

The group redifined topics like "Faculty", which, in the American domain is the teaching staff, i.e. the entire group of teachers. While a "Fakultät" at a German university is an organizational unit for the thematic structure of chairs. A professor in the American domain is a teacher, whereas a German professor is (generally) a holder of a chair.

With this work as a basis, we extended the VIVO ontology with multiple topics through our work in analysing the "Organigrams" (organisational charts) of each member organisation. Structurally the organisational structures would be quite similar, but semantically, the wording, and sometimes the functional meaning of the units would be quite different. To ensure that plurality and local varieties be kept, we needed to take into account that each alliance member would need to use and find research information related to their own terminology. 


To achieve this, e created an upper ontology with common, more abstract classes and an ontology for each member, containing instantiations of common classes retaining local names and idiosyncrasies.	
![Fig 1: Alliance member subsumption hierarchy](images/subsumption_hierarchy_model_extensions_only.png)
*Fig 1: Alliance member subsumption hierarchy*. 

:material-page-next:





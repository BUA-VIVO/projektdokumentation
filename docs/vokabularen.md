# Vocabularies 

The vocabularies used in the Berlin University Alliance (BUA) VIVO project, are used not only to classify publications and other research outputs, but also carry the role of creating a more ample semantic context for the machine readability of the ontologies and the knowledge graphs. The vocabularies and the alliance wide research topic vocabulary, and the assignment of these to outputs as well as projects and institutional structures, form a network bridging semantically diverging entities.

Querying and traversing the knowledge graph, it is possible to make inferences to the similarity of otherwise divergent entities assuming that their semantic likeness can be inferred across them being classified under the same themes and vocabulary topics.

![Fig. 1: Classification of Ontology entities; over-bridging Vocabularies](images/Ontology-Classification.jpg)
*Fig. 1: Classification of Ontology entities; over bridging Vocabularies*.

Fig. 1 is a diagram showing the subsumption hierarchy of the institutional structures, their classification, as well as the relation between vocabularies and projects and research outputs in the BUA knowledge graph
The vocabularies add an additional layer of semantic space for the ontologies and its instances in order to make automatic classification possible. Topics were then extracted from the different vocabularies, defining a semantic space for the vocabulary terms which in themselves are far too generic to be of any use for automatic classification.

## Fächersystematiken des Statistischen Bundesamtes (DESTATIS)
The [Subject classifications of the German Federal Statistical Office](https://www.destatis.de/DE/Methoden/Klassifikationen/Bildung/personal-stellenstatistik.pdf?__blob=publicationFile), is a vocabulary of subject matter groups, intended for the classification of teaching and research entities. It is in our project intended for the classification of institutional entities in our Organization ontologies. In addition, the vocabulary is also the chosen vocabulary for the classification of institutional units, in accordance with the allocation of the organizational units done in the GERiT DFG portal.
We have in the project, created a fork of the [VIVO-de SKOS conversion](https://github.com/VIVO-DE/destatis_faecherklassifikation), and [translated it to English](https://raw.githubusercontent.com/BUA-VIVO/destatis_faecherklassifikation_translation_service/master/faecherklassifikation_skos_en.ttl).

## eudat-b2find Disciplinary Research Vocabulary
The B2FIND is an interdisciplinary  discovery portal for research output. It is a comprehensive joint metadata catalogue and a powerful discovery portal. Metadata stored through EUDAT services such as B2SHARE are harvested from various research communities overarching a wide scope of research disciplines.
Among these datasets, we have chosen the [b2Find research disciplines vocabulary](https://raw.githubusercontent.com/EUDAT-B2FIND/md-ingestion/master/etc/b2find_disciplines.json) as a basis for our disciplinary classification of research outputs.

As part of our project output, we converted the [vocabulary](https://raw.githubusercontent.com/BUA-VIVO/eudat-b2find-skos/main/eudat.ttl) into the [SKOS format](https://www.w3.org/2004/02/skos/), and translated the terms into German.

## Interdisciplinary research field classification
Classification for interdisciplinary research fields is a  Research field classification to support a differentiated representation of interdisciplinary or subject- or problem-related research on the basis of existing research field lists.
The vocabulary was developed in collaboration project between the Humboldt University in Berlin and the German Center for University and Science Research.

As we in the project dealt with research groups having a quite interdisciplinary scope, it was also clear that we would need to use this vocabulary in order to open up for a broader and discipline-overarching classification of research outputs and projects.

In the project, we forked the [kdsf-ffk project](https://github.com/KDSF-FFK/kdsf-ffk/), and created an Englisch translation of it, which  adopted and proof read by the KFID and [published](https://raw.githubusercontent.com/KDSF-FFK/kdsf-ffk/main/FFK.ttl) 
The vocabulary can be viewed at the [SKOSHUB portal](https://skohub.io/KDSF-FFK/kdsf-ffk/heads/main/w3id.org/kdsf-ffk/index.de.html)




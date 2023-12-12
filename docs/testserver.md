# Testserver

The VIVO project used different versions of one or multiple Test-Servers.  
During different stages of the project the following Setups were used:

1. A copy of the VIVO-Server generated as OpenStack Clone  
2. A manually setup VIVO-Server connected to the main Mongo- and SolR-Servers  
3. A newly setup environment of VIVO and SolR-Servers
4. A singular SolR-Testserver
5. A singular MongoDB-Testserver

Server 1. was used in early stages of development for testing frontend changes.  
By replacing the webapps folder with the [live version](https://github.com/BUA-VIVO/vivo-frontend), changes could quickly be tested and exported using git.

Version 2. fulfilled a similar purpose but had the benefits of starting with a clean installation and offered the option of testing the behavior of the live dataset. To test changing datasets, ontologies and frontend changes relying on specific data setup 3. was used.

Testservers 4. and 5. were used to test specific behavior of the installed software, logging and the pipeline behavior during development.

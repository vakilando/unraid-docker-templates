
- [Installation von Docspell unter Unraid mit Docker](#installation-von-docspell-unter-unraid-mit-docker)
  - [Drei mögliche Wege:](#drei-mögliche-wege)
  - [Variante 1: br0](#variante-1-br0)
  - [Variante 2: VLAN](#variante-2-vlan)
  - [Variante 3: Custom Docker Netzwerk](#variante-3-custom-docker-netzwerk)
  - [Vorarbeiten vor der Containererstellung](#vorarbeiten-vor-der-containererstellung)
- [Docspell installation in Unraid with Docker](#docspell-installation-in-unraid-with-docker)
  - [Three possible ways to achieve that:](#three-possible-ways-to-achieve-that)
  - [Variation 1: br0](#variation-1-br0)
  - [Variation 2: VLAN](#variation-2-vlan)
  - [Variation 3: Custom Docker Network](#variation-3-custom-docker-network)
  - [Todo before creating the containers](#todo-before-creating-the-containers)
- [Verwendete Variablen und Konfigurationsdateien<br>Used variables and configuration files](#verwendete-variablen-und-konfigurationsdateienused-variables-and-configuration-files)
  - [Inhalt der .env-Datei <br> Contents of the .env file](#inhalt-der-env-datei--contents-of-the-env-file)
  - [Genutzte Container Variablen in den Templates<br>Used container variables in the templates](#genutzte-container-variablen-in-den-templatesused-container-variables-in-the-templates)

<br>

# Installation von Docspell unter Unraid mit Docker

---

<br>

_Info: Diese Dokumentation is getestet bis Version 0.20.0 von Docspell_  
 
  
Docspell ist ein Dokumentenmanagementsystem (DMS) wie z.B. auch Teedy, Mayan, Paperless, Perpermerge, ...   
Nach langem hin-und-her-testen, habe ich mich für Docspell entschieden.  
Um Docspell unter Unraid zu nutzen müssen mehrere Container installiert werden, die miteinander kommunizieren müssen.
Den Containern müssen diverse Variablen mitgegeben werden und eine Konfigurationsdatei.

Dokumentation/Quellen für Docspell:  
- https://hub.docker.com/r/eikek0/docspell/  
- https://github.com/eikek/docspell
- https://docspell.org/docs/

Thread im Unraid Forum:
- https://forums.unraid.net/topic/103425-docspell-hilfe/

<br>  

## Drei mögliche Wege:
-   **br0**  
_Vorteil_: Jeder Container erhält eine IP, somit muss man sich bzgl. evtl. bereits verwendeter Ports weniger Gedanken machen (siehe unten).  
 Diese Variante ist am einfachsten...

-   **VLAN**  
_Vorteil_: Jeder Container erhält eine IP aus dem VLAN, somit muss man sich bzgl. evtl. bereits verwendeter Ports weniger Gedanken machen (siehe unten). Zudem kann jeder Container dank eigener IP besser über eine Firewall "kontrolliert" werden.

-   **Custom Docker Netzwerk**  
_Vorteil_: Die Kommunikation zwischen den Containern kann über deren Namen erfolgen (siehe z.B. unten bei "_docspell.conf_").
    
Ich habe anfangs ein "_custom docker network_" verwendet, habe dann aber zugunsten der eigenen IP-Adressen je Container auf ein VLAN gewechselt, zumal ich eh VLANs verwende.   
  
<br>  


## Variante 1: br0
- Einfach beim anlegen der Docker Container anstelle von "_bridge_"  "_br0_" verwenden
- Mit "_br0_" kann jedem Container eine eigene IP aus der selben Range wie dem Unraid Server zugewiesen werden.   

<br>  

## Variante 2: VLAN

- Damit VLANs verwendet werden können muss der verwendete Router/Switch VLANs unterstützen!
- in Unraid unter "_Settings_" => "_Network Settings_" => "_Enable VLANs_" auf "_Yes_" stellen und die notwendigen Einstellungen vornehmen (Docker- und VM-Dienst muss vorher beendet werden).
- Das neu erstellte VLAN (z.B. "_br0.5_") kann nun bei der Container Erstellung ausgewählt werden.

<br>  

## Variante 3: Custom Docker Netzwerk

- Damit das Netzwerk bei der Erstellung der Container verfügbar ist muss in Unraid zuerst unter "_Settings_" => "_Docker_" 
der Punkt "_Preserve user defined networks_" auf "_Yes_" gestelt werden (Docker Dienst muss vorher beendet werden).
- Nun per SSH oder über die Unraid GUI auf die Console des Servers wechseln.  
- Ein Custum Docker Netzwerk erstellen:  
  Einfach:    `docker network create dnet-docspell`  
  Erweitert: `docker network create --driver=bridge --subnet=192.168.3.0/25 --gateway=192.168.3.1  dnet-docspell`  
- Das neu erstellte custum network "_dnet-docspell_" kann nun bei der Container Erstellung ausgewählt werden

<br>  

## Vorarbeiten vor der Containererstellung

1. **Verzeichnisse anlegen**  
   Verzeichnis "`docspell`" unter "`appdata`" anlegen.  
   Verzeichnis "`docspell/opt`" unter "`appdata`" anlegen.  

2. **Konfigurationsdatei anlegen**  
   "`docspell.conf`" in "`/docspell/opt`" anlegen.  
   Es kann die Datei von github genommen werden: <https://github.com/eikek/docspell/blob/master/docker/docspell.conf>  
     
   Ich habe folgende Änderungen vorgenommen:  
   - alle Dockernamen durch ihre IP-Adresse ersetzt  
     also z.B. "`http://joex:7878`"  
     => "`http://192.168.3.1:7878`"  
   - alle Variablen durch ihre Werte ersetzt  
     also z.B. "`jdbc:"\${DB_TYPE}"://"\${DB_HOST}":"\${DB_PORT}"/"\${DB_NAME}` => `"jdbc:postgresql://192.168.3.2:5432/dbname"`  
       
   Ich habe es NICHT getestet aber:  
   - bei Verwendung eines _custom Docker Netzwerks_ sollte die Dockernamensauflösung funktionieren...  
   - da die Variablen in den Unraid Docker Templates enthalten sind sollten die Variablen hier funktionieren...  

3. **Gemeinsam genutzte Variablen**  
   Die Docker Container benötigen gemeinsame Variablen (siehe "_docspell.conf_").  
   Die Variablen werden  
   - **ENTWEDER** in jedem Telmplate angepasst  
     (Die Variablen sind dort vordefiniert und beschrieben)
   - **ODER** über eine zentrale Datei unter "_Extra Parameters_" angegeben:  
   "`--env-file=/mnt/user/appdata/docspell/.env`"  
      Es kann die Datei von github genommen werden: <https://github.com/eikek/docspell/blob/master/docker/.env>  
      Sie wird unter "`../appdata/docspell`" gespeichert.  
        
      Die "`.env`"-Datei ist praktisch um Vorgaben zentral zu speichern, erfordert aber das Anlegen der Datei _vor_ Erstellung der Docker.  

4. **Unraid Templates einbinden**  
   In Unraid unter "_Docker_" => "_Template Repositories_" (ganz unten) folgende URL eintragen und auf "_Save_" klicken: <https://github.com/vakilando/unraid-docker-templates>  

5. **Container einrichten**  
   In Unraid unter "_Docker_" unten auf "**ADD CONTAINER**" klicken.  
   Wähle die Container "_vakilando-docspell-xxx_" aus  
   Siehe unten für weitere Informationen bzgl. der Templates.     
   <br>
   1. **postgres**   
      Ich verwende Postgresql aus dem original Repository  
      Repository: postgres:11.7  
      Docker: https://hub.docker.com/\_/postgres/  
        
      _**Wichtige Einstellungen:** sind **fett** markiert._  
      Sie müssen bei jedem Container identisch sein und mit Angaben in der "_docspell.conf_" übereinstimmen.

      <br>

      Feld | Wert  (anzupassen!)  
      ---------|----------
       Repository: | postgres:11.7
       Network Type: | custom-network oder VLAN wählen
       Fixed IP address:  | ...sofern gewünscht...
       Extra Parameters |**--hostname db**
       POSTGRES_PASSWORD | **somesafepassword**
       POSTGRES_USER | **dbuser**
       POSTGRES_DB | **dbname**
       Database Storage Path | /mnt/cache/appdata/postgres/
       Web Interface Port | **5432**
      <br>    

   2. **solr**   
      Ich verwende Solr aus dem Repository "_A75G's Repository_"  
      Docker: <https://hub.docker.com/r/bitnami/solr/>  
      Support: https://forums.unraid.net/topic/89502-support-a75g-repo/  
      Project: https://lucene.apache.org/solr/  
        
      _**Wichtige Einstellungen** sind **fett** markiert._  
      Die Variable "_SOLR_CORE_" wird in der "_docspell.conf_" verwendet (solr URL).

      Feld | Wert  (anzupassen!)  
      ---------|----------
       Repository: | bitnami/solr:8
       Network Type: | custom-network oder VLAN wählen
       Fixed IP address:  | ...sofern gewünscht...
       Port: | **8983**
       Appdata: | z.B. /mnt/user/appdata/docspell/solr_data
       SOLR_PORT_NUMBER: | **8983**
       SOLR_CORE: | **docspell**
      <br>    
   
   3. **vakilando-docspell-joex**  
      
      _**Wichtige Einstellungen:** sind **fett** markiert._  
      Sie müssen bei jedem Container identisch sein und mit Angaben in der "_docspell.conf_" übereinstimmen.
        
      <br>
        
      Feld | Wert 
      ---------|----------
       Repository: | eikek0/docspell:joex-LATEST
       Network Type: | custom-network oder VLAN wählen
       Fixed IP address:  | ...sofern gewünscht...
       Host Port 1: | **7878**
       docspell.conf | /mnt/user/appdata/docspell/opt/docspell.conf<br>(_muss vorhanden sein!!_)
       TZ | Europe/Berlin
       DOCSPELL_HEADER_VALUE | **SomeRandomString**  
       DB_TYPE | postgresql
       DB_HOST | Name des DB-Containers z.B. "**db**"<br> oder IP-Adresse "**192.168.3.2**"
       DB_PORT | **5432**
       DB_NAME | **dbname**  
       DB_USER | **dbuser**  
       DB_PASS | **somesafepassword**  
      <br>    
   
   4. **vakilando-docspell-restserver**

      _**Wichtige Einstellungen:** sind **fett** markiert._  
      Sie müssen bei jedem Container identisch sein und mit Angaben in der "_docspell.conf_" übereinstimmen.
        
      <br>
        
      Feld | Wert 
      ---------|----------
       Repository: | eikek0/docspell:restserver-LATEST
       Network Type: | custom-network oder VLAN wählen
       Fixed IP address:  | ...sofern gewünscht...
       Host Port 1: | **7880**
       docspell.conf | /mnt/user/appdata/docspell/opt/docspell.conf<br>(_muss vorhanden sein!!_)
       TZ | Europe/Berlin
       DOCSPELL_HEADER_VALUE | **SomeRandomString**  
       DB_TYPE | postgresql
       DB_HOST | Name des DB-Containers z.B. "**db**"<br> oder IP-Adresse "**192.168.3.2**"
       DB_PORT | **5432**
       DB_NAME | **dbname**  
       DB_USER | **dbuser**  
       DB_PASS | **somesafepassword**  
      <br>    

   5. **vakilando-docspell-consumedir**  
      
      _**Wichtige Einstellungen:** sind **fett** markiert._  
      Sie müssen bei jedem Container identisch sein und mit Angaben in der "_docspell.conf_" übereinstimmen.
        
      <br>
        
      Feld | Wert (anzupassen!)  
      ---------|----------
       Repository: | eikek0/docspell:consumedir-LATEST
       Network Type: | custom-network oder VLAN wählen
       Fixed IP address:  | ...sofern gewünscht...
       Consumedir: | z.B. /mnt/user/appdata/docspell/docs
       TZ | Europe/Berlin
       DOCSPELL_HEADER_VALUE | **SomeRandomString**  
       DB_TYPE | postgresql
       DB_HOST | Name des DB-Containers z.B. "**db**"<br> oder IP-Adresse "**192.168.3.2**"
       DB_PORT | **5432**
       DB_NAME | **dbname**  
       DB_USER | **dbuser**  
       DB_PASS | **somesafepassword**  
       CONSUMEDIR_INTEGRATION | **y**
      <br>    

6. **Die Reihenfolge in der die Container starten ist wichtig!**  
   Daher ist diese nach der Installation durch verschieben mit der Maus ggf. zu korrigieren:  
    1. postgres  
    2. solr  
    3. docspell-joex  
    4. docspell-restserver  
    5. docspell-consumedir  

<br><br>


------------  
------------  
<br>

# Docspell installation in Unraid with Docker

_Info: This documentation is tested up to version 0.20.0 of docspell_  
 
Docspell is a personal document organizer / management system (DMS) like Teedy, Mayan, Paperless, Perpermerge,...  
To run Docspell under unraid as docker, several containers have to be installed that have to communicate with each other.  
Various variables and a configuration file must be given to the containers.

Documentation/Sources for Docspell:  
- https://hub.docker.com/r/eikek0/docspell/  
- https://github.com/eikek/docspell
- https://docspell.org/docs/

Thread in the Unraid Forum:
- https://forums.unraid.net/topic/103425-docspell-hilfe/

<br>  

## Three possible ways to achieve that:
- **br0**  
_Advantage_: Each container receives an IP from the IP-range your unraid server is in, so you don't have to worry about any ports that may already be used (see below).  
 Diese Variante ist am einfachsten...
- **VLAN**  
_Advantage_: Each container receives an IP from the VLAN, so you don't have to worry about any ports that may already be used (see below). In addition, each container can be better "controlled" via a firewall thanks to its own IP.
- **Custom Docker Network**  
_Advantage_: The communication between the containers can take place via their names (see e.g. below under "_docspell.conf_").
    
At the beginning I used a "_custom docker network_", but then switched to a VLAN in favor of the own IP addresses for each container and beeing able to better control the container communication with my firewall and switches.  
<br>  


## Variation 1: br0

- When configuring the docker container use "_br0_" and not "_bridge_"
- Using "_br0_" each container has its own IP address that is in the IP range the Unraid Server is in.   

<br>  

## Variation 2: VLAN

- To use VLANs the router / switch used must support VLANs!
- In Unraid go to "_Settings_" => "_Network Settings_" => "_Enable VLANs_", set it to "_Yes_" and make the necessary settings (Docker and VM service must be terminated beforehand).
- The newly created VLAN (e.g. "_br0.5_") can now be selected when creating the container.

<br>  

## Variation 3: Custom Docker Network

- In order for the network to be available when creating the container:  
  In Unraid go to "_Settings_" => "_Docker_". Choose the item "_Preserve user defined networks_" and set it to "_Yes_" (Docker service must be terminated beforehand).
- Now switch to the server console via SSH or the Unraid GUI.
- Create a Custum Docker network:  
  Simple: `docker network create dnet-docspell`  
  Extended: `docker network create --driver = bridge --subnet = 192.168.3.0 / 25 --gateway = 192.168.3.1 dnet-docspell`  
- The newly created custom network "_dnet-docspell_" can now be selected when creating the container.  

<br>  

## Todo before creating the containers

1. **Create directories**  
   Create the directory "`docspell`" in "`appdata`".  
   Create the directory "`docspell/opt`" in "`appdata`".  

2. **Create configuration files**  
   Create the file "`docspell.conf`" in "`/docspell/opt`".  
   The file can be taken from github: <https://github.com/eikek/docspell/blob/master/docker/docspell.conf>
     
   I made the following changes:
   - all docker names replaced by their IP address:  
     e.g.: "`http://joex:7878`" => "`http://192.168.3.1:7878`"  
   - replaced all variables with their values:  
     e.g. "`jdbc:"\${DB_TYPE}"://"\${DB_HOST}":"\${DB_PORT}"/"\${DB_NAME}` => `"jdbc:postgresql://192.168.3.2:5432/dbname"`  
       
   I have NOT tested but:
   - when using a _custom Docker network_ the Docker name resolution should work (no need to replace names by IP's)...
   - since the variables are contained in the Unraid Docker templates, the variables should work here too (no need to replace them)...  
  
3. **Common used variables**  
   The Docker containers need common variables (see "_docspell.conf_").  
   The variables are  
   - EITHER individually adapted and used in the telmplates  
     (The variables are already specified and described there)
   - OR specified via a central file under "_Extra Parameters_":  
   "`--env-file=/mnt/user/appdata/docspell/.env`"  
      The file can be taken from github: <https://github.com/eikek/docspell/blob/master/docker/.env>  
      It is saved under "`../appdata/docspell`".  
        
      The "_.env_" file is useful for storing specifications centrally, but requires the file to be created before the Docker is created.  

4. **Using the Unraid templates**  
   In Unraid got to "_Docker_" => "_Template Repositories_" (at the bottom) and add this URL: <https://github.com/vakilando/unraid-docker-templates>  
   Click "_Save_"

5. **Install th containers**  
   In Unraid go to "_Docker_" and at the bottom click on "**ADD CONTAINER**".  
   Choose the Containers "_vakilando-docspell-xxx_"  
   See above or beneath for more instructions concerning the templates..  
   <br>

6. **Change the start sequence of the containers!**  
    1. postgres  
    2. solr  
    3. docspell-joex  
    4. docspell-restserver  
    5. docspell-consumedir  

<br>  
<br>  

-------------  
-------------  

<br>  
<br>  

# Verwendete Variablen und Konfigurationsdateien<br>Used variables and configuration files  


## Inhalt der .env-Datei <br> Contents of the .env file  

----
TZ=Europe/Berlin  
DOCSPELL_HEADER_VALUE=SomeRandomString  
DB_TYPE=postgresql  
DB_HOST=db  
DB_PORT=5432  
DB_NAME=dbname  
DB_USER=dbuser  
DB_PASS=dbpass    

<br>

## Genutzte Container Variablen in den Templates<br>Used container variables in the templates

_____________________

Feld | Wert 
---------|----------
Name: | TZ  
Key: | TZ  
Value: | Europe/Berlin  
Default Value: | Europe/Berlin  
Description: | TZ=Europe/Berlin<br>Timezone to set  
<br>  
 
_____________________

Feld | Wert 
---------|----------
Name: | DOCSPELL_HEADER_VALUE  
Key: | DOCSPELL_HEADER_VALUE  
Value: | SomeRandomString  
Default Value: |   
Description: | DOCSPELL_HEADER_VALUE=SomeRandomString<br> defines a secret that is shared between some containers.<br>Please see the consumedir.sh docs for additional info.  

<br>  
  
  
----  
  
Feld | Wert  
---------|----------
Name: | DB_TYPE  
Key: | DB_TYPE  
Value: | postgresql  
Default Value: | postgresql  
Description: | DB_TYPE=postgresql<br>type of database to use (postgresql, mariadb, h2).<br>You need an allready running database server  
<br>  
  
_____________________

Feld | Wert 
---------|----------
Name: | DB_HOST  
Key: | DB_HOST  
Value: | db  
Default Value: | db  
Description: | DB_HOST=db<br>database host name or IP 
<br>
  
_____________________

Feld | Wert 
---------|----------
Name: | DB_PORT  
Key: | DB_PORT  
Value: | 5432  
Default Value: | 5432  
Description: | DB_PORT=5432  (postgresql)<br>database host port  
<br>

_____________________

Feld | Wert 
---------|----------
Name: | DB_NAME  
Key: | DB_NAME  
Value: | dbname  
Default Value: | dbname  
Description: | DB_NAME=dbname<br>database name. the database will be created at first run  
<br>

_____________________

Feld | Wert 
---------|----------
Name: | DB_USER  
Key: | DB_USER  
Value: | dbuser  
Default Value: | dbuser  
Description: | DB_USER=dbuser<br>database user name  
<br>


_____________________

Feld | Wert 
---------|----------
Name: | DB_PASS  
Key: | DB_PASS  
Value: | somesafepassword  
Default Value: | somesafepassword  
Description: | DB_PASS=somesafepassword<br>database password  
  
  
  



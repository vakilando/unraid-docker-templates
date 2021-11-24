
- [Hinweis: Änderungen ab docspell v0.24](#hinweis-änderungen-ab-docspell-v024)
    - [Die Hauptunterschiede:](#die-hauptunterschiede)
  - [------- *IN BEARBEITUNG - WORK in PROGRESS* -------](#--------in-bearbeitung---work-in-progress--------)
- [Installation von Docspell in Unraid Docker](#installation-von-docspell-in-unraid-docker)
  - [NETZWERK: Drei Möglichkeiten](#netzwerk-drei-möglichkeiten)
  - [DOCKER CONTAINER: Vorarbeiten & Erstellung](#docker-container-vorarbeiten--erstellung)
- [Installation of Docspell in Unraid with Docker](#installation-of-docspell-in-unraid-with-docker)
  - [NETWORK: Three possible ways](#network-three-possible-ways)
  - [DOCKER CONTAINER: Todo and creating](#docker-container-todo-and-creating)
- [Variablen und Konfigurationsdateien<br>Variables and configuration files](#variablen-und-konfigurationsdateienvariables-and-configuration-files)
  - [Inhalt der .env-Datei <br> Contents of the .env file](#inhalt-der-env-datei--contents-of-the-env-file)
  - [Genutzte Container Variablen in den Templates<br>Used container variables in the templates](#genutzte-container-variablen-in-den-templatesused-container-variables-in-the-templates)


<br>  
<br>  

# Hinweis: Änderungen ab docspell v0.24
Bis Version 0.23 funktioniert die alte Version [Reamde hier](README_docspell-v0.23.md "Readme Docspell v0.23").  
Ab Version 0.25 (vermutlich schon ab v0.24) muss diese Vorgehensweise verwendet werden.  
Warum nicht ab v0.24? Ich bin von v0.23 direkt auf die v0.25, da ich die Änderungen von ekek0 schlicht nicht mibekonnen habe.  
### Die Hauptunterschiede:  
- Wechsel des Repository: von "`eikek0/docspell`" zu "`docspell`"
- Wechsel von Postgresql: von 11.7 zu Postgresql 13.3  
_Achtung:_ kein einfaches Upgrade über Major Versionen möglich (siehe Beschreibung unten)!
- Erneut Benutzung der `.env` Datei für globale Variablen

<br>  
<br>  

## ------- *IN BEARBEITUNG - WORK in PROGRESS* -------
TODO:  
- Aktualisierung der Container Variablen.  
Die Templates und die Installation funktionieren zwar, aber ich denke ich kann die Angabe der Variablen und mitgegebenen Docker commands (in "Post Arguments") noch *optimieren*  

DONE :
- Aktualisierung der Templates (v0.23 > v0.25)!  
Installation funktioniert mit hier angegebenen Variablen und mitgegebenen Docker commands (in "Post Arguments").  
<br>  
<br>  

# Installation von Docspell in Unraid Docker


_Info: Diese Dokumentation is getestet bis Version 0.27 von Docspell_  
 
  
Docspell ist ein Dokumentenmanagementsystem (DMS).  
Getestet habe ich auch Teedy, Mayan, Paperless, Perpermerge.  
Aber Docspell hat mich überzeugt. Es ist einfach gehalten und übersichtlich.  
  
In Unraid müssen mehrere Container für Doscspell installiert werden, die untereinander kommunizieren müssen.  
Den Containern müssen diverse Variablen mitgegeben werden.  
Zu den Variablen gehören auch die Pfade der Dateien `docspell.conf`und `.env`.  
Diese Dateien müssen _<b>vor</b>_ dem (ersten) Start des Dockers vorhanden sein!.  

Dokumentation/Quellen für Docspell:  
- https://hub.docker.com/r/eikek0/docspell/  
- https://github.com/eikek/docspell
- https://docspell.org/docs/

Thread im Unraid Forum:
- https://forums.unraid.net/topic/103425-docspell-hilfe/

<br>  
<br>  

## NETZWERK: Drei Möglichkeiten
**br0**  
  
_Vorteil_:  
Jeder Container erhält eine eigene IP. Über die Vergabe von Ports muss man sich so weniger Gedanken machen (siehe unten). Zudem kann jeder Container dank eigener IP besser über eine Firewall "kontrolliert" werden.  
Diese Variante ist am einfachsten...
<br>  
_Vorgehen:_  

 - Beim anlegen der Docker Container als Netzwerk `br0` verwenden (anstelle von `bridge`)  
 - Mit "br0" kann jedem Container eine eigene IP aus der selben Range wie dem Unraid Server zugewiesen werden.


**VLan**  
  
_Vorteil_:  
Jeder Container erhält eine eigene IP aus dem verwendetem VLAN. Über die Vergabe von Ports muss man sich so weniger Gedanken machen (siehe unten). Zudem kann jeder Container dank eigener IP besser über eine Firewall "kontrolliert" werden.

- Damit VLANs verwendet werden können muss der verwendete Router/Switch VLANs unterstützen!
- in Unraid unter "_Settings_" => "_Network Settings_" => "_Enable VLANs_" auf "_Yes_" stellen und die notwendigen Einstellungen vornehmen (Docker- und VM-Dienst muss vorher beendet werden).
- Das neu erstellte VLAN (z.B. "_br0.5_") kann nun bei der Container Erstellung ausgewählt werden.

**Custom Docker Netzwerk**  
  
_Vorteil_:  
Die Kommunikation zwischen den Containern kann über deren Namen erfolgen (siehe z.B. unten bei "_docspell.conf_").
    
Ich habe anfangs ein "_custom docker network_" verwendet, habe dann aber zugunsten der eigenen IP-Adressen je Container auf ein VLAN gewechselt, zumal ich eh VLANs verwende.   
  

- Damit das Netzwerk bei der Erstellung der Container verfügbar ist muss in Unraid zuerst unter "_Settings_" => "_Docker_" 
der Punkt "_Preserve user defined networks_" auf "_Yes_" gestelt werden (Docker Dienst muss vorher beendet werden).
- Nun per SSH oder über die Unraid GUI auf die Console des Servers wechseln.  
- Ein Custum Docker Netzwerk erstellen:  
  Einfach:    `docker network create dnet-docspell`  
  Erweitert: `docker network create --driver=bridge --subnet=192.168.3.0/25 --gateway=192.168.3.1  dnet-docspell`  
- Das neu erstellte custum network "_dnet-docspell_" kann nun bei der Container Erstellung ausgewählt werden

<br>  
<br>  


## DOCKER CONTAINER: Vorarbeiten & Erstellung

1. **Verzeichnisse anlegen**  
   Verzeichnis "`docspell`" unter "`appdata`" anlegen.  
   Verzeichnis "`docspell/opt`" unter "`appdata`" anlegen.  

2. **Konfigurationsdatei anlegen**  
   "`docspell.conf`" in "`/docspell/opt`" anlegen.  
   Es kann die Datei von github genommen werden: <https://github.com/eikek/docspell/blob/master/docker/docker-compose/docspell.conf>  
     
   Ich habe folgende Änderungen vorgenommen:  
   - alle Dockernamen durch ihre IP-Adresse ersetzt  
     also z.B. "`http://joex:7878`"  
     => "`http://192.168.3.1:7878`"  
   - alle Variablen durch ihre Werte ersetzt  
     also z.B. "`jdbc:"\${DB_TYPE}"://"\${DB_HOST}":"\${DB_PORT}"/"\${DB_NAME}` => `"jdbc:postgresql://192.168.3.2:5432/dbname"`  
       
   Ich habe es NICHT getestet aber:  
   Wird ein *custom Docker Network* verwendet, sollte die Namensauflösung funktionieren und eine Ersetzung der Dockernamen durch ihre IP-Adresse sollte nicht notwendig sein.  
   Da die Variablen in den Docker Templates und der `.env`Datei (wenn verwendet) enthalten sind, sollten sie hier funktionieren (Ersetzen nicht notwendig).  

3. **Variablen**  
   Die Docspell Container benötigen selbst und gemeinsame genutzte Variablen (siehe auch in "_docspell.conf_").
   Die Variablen können in jedem Telmplate angepasst werden (Die Variablen sind dort vordefiniert und beschrieben).  
   Für gemeinsam genutzte Variablen die kann die Verwendung einer zentralen Datei `.env` praktisch sein um Vorgaben zentral zu speichern, erfordert aber das Anlegen der Datei _vor_ Erstellung der Docker unter "`../appdata/docspell`".  
   Sie wird unter "*Extra Parameters*" angegeben:  
   "`--env-file=/mnt/user/appdata/docspell/.env`"  
      Es kann die Datei von github genommen werden: <https://github.com/eikek/docspell/blob/master/docker/docker-compose/.env>  
        

4. **Unraid Templates einbinden**  
   In Unraid unter "_Docker_" => "_Template Repositories_" (ganz unten) folgende URL eintragen und auf "_Save_" klicken: <https://github.com/vakilando/unraid-docker-templates>  

5. **Postgresql upgrade!**  
   Wenn bereits eine ältere Postgresdatenbank mit docspell vorhanden ist sollte sie ein Upgrade erfahren.  
   Ein Upgrade von Version 11.7 auf 13.3 wird hier beschrieben:  
    1. **Postgresql 13.3 in Unraid parallel zu 11.7 installieren (official repository)**  
       Der neue Postgresql 13.3 bekommt die selben Einstellungen wie der 11.7 nur:  
        - einen anderen Namen (sonst ist eine parallele Installation nicht möglich)
        - eine andere IP (temporär..., sonst ist ein paraleller Start nicht möglich)
        - eine andere MAC Adresse (temporär..., sonst ist ein paraleller Start nicht möglich)

   2. **BACKUP der alte Postgresql 11.7 Datenbank JETZT**  
      Auf Unraid, Backup Verzeichnis in appdata von postgres 11.7 erstellen und betreten:  
      `mkdir /mnt/cache/appdata/postgres-11.7/BACKUPS`  
      `cd /mnt/cache/appdata/postgres-11.7/BACKUPS`  
      Für eine Vollsicherung des ganzen DB-Servers:  
      `docker exec postgres-11.7 pg_dumpall -U pgadmin > pg_dumpall_pg11.7.bkp`  
      Für eine Sicherung nur der Datenbank "docspell":  
      `docker exec postgres-11.7 pg_dump -U pgadmin -Fc docspell > pg_dump_docspell_pg11.7.bkp`  

   3. **Importieren des Postgresql 11.7 Datenbank Backups in Postgresql 13.3**  
      Auf Unraid, Backup Verzeichnis in appdata von postgres 13.3  erstellen und betreten:  
      `mkdir /mnt/cache/appdata/postgres-13.3/BACKUPS`  
      `cd /mnt/cache/appdata/postgres-13.3/BACKUPS`  
      Kopieren der 11.7 Backups nach appdata des neuen Postgresql 13.3  
      Vollsicherung: `cp /mnt/cache/appdata/postgres-11.7/BACKUPS/pg_dumpall_pg11.7.bkp .`  
      Datenbank: `cp /mnt/cache/appdata/postgres-11.7/BACKUPS/dpg_dump_docspell_pg11.7.bkp .`  
      _Vollsicherung zurückspielen:_  
      Die Console des Postgresql 13.3 Container starten und in das Backup Verzeichnis wechseln:  
      `cd /var/lib/postgresql/data/BACKUPS`  
      Vollsicherung zurückspielen:  
      `psql -U pgadmin -f pg_dumpall_pg11.7_2.bkp postgres`  
      _Datenbanksicherung zurückspielen:_  
      Über das Unraid "Webterminal" oder über SSH (z.B. mit Putty bei Unraid anmelden):
      `docker exec -i -u pgadmin postgres-13.3 pg_restore -C -d postgres < dpg_dump_docspell_pg11.7.bkp`  

5. **Container anpassen**  
Alter Container postgresql-11.7: Autostart AUS  
Neuer Container postgresql-13.3: IP- und MAC-Adresse auf die Werte des alten setzen und Autostart AN  

6. **Container einrichten**  
   In Unraid unter "_Docker_" unten auf "**ADD CONTAINER**" klicken.  
   Wähle die Container "_vakilando-docspell-xxx_" aus  
   Siehe unten für weitere Informationen bzgl. der Templates.     
   <br>
   1. **postgres**   
      Ich verwende Postgresql aus dem original Repository.  
      Verwendet wird die Version wie in der dockspell docker-compose.yml: postgres:13.3.  
      Ich habe bereits eine Installation mit Postgres:11.7, also ist ein Upgrade notwendig: siehe oben.
      Docker: https://hub.docker.com/\_/postgres/  
        
      _**Wichtige Einstellungen:** sind **fett** markiert._  
      Sie müssen bei jedem Container identisch sein und mit Angaben in der "_docspell.conf_" übereinstimmen.

      <br>

      Feld | Wert  (anzupassen!)  
      ---------|----------
       Repository: | postgres:13.3
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
      Ich verwende Solr aus dem Repository "bitnami"  
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
       Appdata: | z.B. /mnt/user/appdata/solr_data
       SOLR_PORT_NUMBER: | **8983**
       SOLR_CORE: | **docspell**
      <br>    
   
   3. **vakilando-docspell-joex**  
      
      _**Wichtige Einstellungen:** sind **fett** markiert._  
      Sie müssen bei jedem Container identisch sein und mit Angaben in der "_docspell.conf_" übereinstimmen.
        
      <br>
        
      Feld | Wert 
      ---------|----------
       Repository: | docspell/joex:latest
       Extra Parameters (NEU) | --mac-address 04:32:C1:12:02:13 --env-file=/mnt/user/appdata/docspell/.env -e JAVA_OPTS="-Xmx2500m" --no-healthcheck
       Erklärung: | --mac-address: optional<br>-e JAVA_OPTS: optional (mehr RAM für Java)
       Post Arguments: (NEU) | /opt/docspell.conf
       Network Type: | br0, custom-network oder VLAN wählen
       Fixed IP address:  | ...sofern gewünscht...
       Host Port 1: | **7878**
       /opt/docspell.conf | /mnt/user/appdata/docspell/opt/docspell.conf<br>(_muss vorhanden sein!!_)
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
       Repository: | docspell/restserver:latest
       Extra Parameters (NEU) | --mac-address 04:42:E2:13:03:11 --env-file=/mnt/user/appdata/docspell/.env --hostname docspell.domain.com
       Erklärung: | --mac-address: optional
       Post Arguments: (NEU) | /opt/docspell.conf
       Network Type: | custom-network oder VLAN wählen
       Fixed IP address:  | ...sofern gewünscht...
       Host Port 1: | **7880**
       /opt/docspell.conf | /mnt/user/appdata/docspell/opt/docspell.conf<br>(_muss vorhanden sein!!_)
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
       Repository: | docspell/dsc:latest
       Extra Parameters (NEU) | --mac-address 22:42:E1:55:21:03 --env-file=/mnt/user/appdata/docspell/.env --restart=unless-stopped
       Erklärung: | --mac-address: optional
       Post Arguments: (NEU) | dsc "-vv" "-d" "http://ip.add.re.ss:7880" "watch" "--delete" "-ir" "--header" "Docspell-Integration:SomeRandomString" "/opt/docs"
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

7. **Die Reihenfolge in der die Container starten ist wichtig!**  
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

# Installation of Docspell in Unraid with Docker

_Info: This documentation is tested up to version 0.25.1 of docspell_  
 
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

## NETWORK: Three possible ways
**br0**  
_Advantage_: Each container gets an IP from the IP-range your unraid server is in. You don't have to worry about any ports that may already be used (see below). In addition, each container can be better "controlled" via a firewall thanks to its own IP.

- When configuring the docker container use "_br0_" and not "_bridge_"
- Using "_br0_" each container has its own IP address that is in the IP range the Unraid Server is in.   

**VLAN**  
_Advantage_: Each container gets an IP from the VLAN IP-range. You don't have to worry about any ports that may already be used (see below). In addition, each container can be better "controlled" via a firewall thanks to its own IP.

- To use VLANs the router / switch used must support VLANs!
- In Unraid go to "_Settings_" => "_Network Settings_" => "_Enable VLANs_", set it to "_Yes_" and make the necessary settings (Docker and VM service must be terminated beforehand).
- The newly created VLAN (e.g. "_br0.5_") can now be selected when creating the container.

**Custom Docker Network**  
_Advantage_: The communication between the containers can take place via their names (see e.g. below under "_docspell.conf_").
    
At the beginning I used a "_custom docker network_", but then switched to a VLAN in favor of the own IP addresses for each container and beeing able to better control the container communication with my firewall and switches.  

- In order for the network to be available when creating the container:  
  In Unraid go to "_Settings_" => "_Docker_". Choose the item "_Preserve user defined networks_" and set it to "_Yes_" (Docker service must be terminated beforehand).
- Now switch to the server console via SSH or the Unraid GUI.
- Create a Custum Docker network:  
  Simple: `docker network create dnet-docspell`  
  Extended: `docker network create --driver = bridge --subnet = 192.168.3.0 / 25 --gateway = 192.168.3.1 dnet-docspell`  
- The newly created custom network "_dnet-docspell_" can now be selected when creating the container.  

<br>  
<br>  

## DOCKER CONTAINER: Todo and creating

1. **Create directories**  
   Create the directory "`docspell`" in "`appdata`".  
   Create the directory "`docspell/opt`" in "`appdata`".  

2. **Create configuration files**  
   Create the file "`docspell.conf`" in "`/docspell/opt`".  
   The file can be taken from github: <https://github.com/eikek/docspell/blob/master/docker/docker-compose/docspell.conf>
     
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
      The file can be taken from github: <https://github.com/eikek/docspell/blob/master/docker/docker-compose/.env>  
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

# Variablen und Konfigurationsdateien<br>Variables and configuration files  


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
  
  
  



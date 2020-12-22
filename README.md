# Installation von Docspell unter Unraid mit Docker

- [Installation von Docspell unter Unraid mit Docker](#installation-von-docspell-unter-unraid-mit-docker)
  - [Zwei mögliche Wege:](#zwei-mögliche-wege)
  - [Variante 1: Custom Docker Netzwerk](#variante-1-custom-docker-netzwerk)
  - [Variante 2: VLAN](#variante-2-vlan)
  - [Vorarbeiten vor der Containererstellung](#vorarbeiten-vor-der-containererstellung)
  - [## Inhalt der .env-Datei](#-inhalt-der-env-datei)
  - [Genutzte Container Variablen in den Templates](#genutzte-container-variablen-in-den-templates)

<br>

---

<br>

Docspell ist ein Dokumentenmanagementsystem (DMS) wie z.B. auch Teedy, Mayan, Paperless, Perpermerge, ...   
Nach langem hin-und-her-testen, habe ich mich für Docspell entschieden.  
Um Docspell unter Unraid zu nutzen müssen mehrere Container installiert werden, die miteinander kommunizieren müssen.
Den Containern müssen diverse Variablen mitgegeben werden und eine Konfigurationsdatei.

Dokumentation/Quellen für Docspell:  
- https://hub.docker.com/r/eikek0/docspell/  
- https://github.com/eikek/docspell
- https://docspell.org/docs/


<br>  

## Zwei mögliche Wege:
-   **Custom Docker Netzwerk**  
_Vorteil_: Die Kommunikation zwischen den Containern kann über deren Namen erfolgen (siehe z.B. unten bei "_docspell.conf_").
-   **VLAN**  
_Vorteil_: Jeder Container erhält eine IP aus dem VLAN, somit muss man sich bzgl. evtl. bereits verwendeter Ports weniger Gedanken machen (siehe unten). Zudem kann jeder Container dank eigener IP besser über eine Firewall "kontrolliert" werden.
    
Ich habe anfangs ein "_custom docker network_" verwendet, habe dann aber zugunsten der eigenen IP-Adressen je Container auf ein VLAN gewechselt.   
  
<br>  


## Variante 1: Custom Docker Netzwerk

- Damit das Netzwerk bei der Erstellung der Container verfügbar ist muss in Unraid zuerst unter "_Settings_" => "_Docker_" 
der Punkt "_Preserve user defined networks_" auf "_Yes_" gestelt werden (Docker Dienst muss vorher beendet werden).
- Nun per SSH oder über die Unraid GUI auf die Console des Servers wechseln.  
- Ein Custum Docker Netzwerk erstellen:  
  Einfach:    `docker network create dnet-docspell`  
  Erweitert: `docker network create --driver=bridge --subnet=192.168.3.0/25 --gateway=192.168.3.1  dnet-docspell`  
- Das neu erstellte custum network "_dnet-docspell_" kann nun bei der Container Erstellung ausgewählt werden

<br>  

## Variante 2: VLAN

- Damit VLANs verwendet werden können muss der verwendete Router/Switch VLANs unterstützen!
- in Unraid unter "_Settings_" => "_Network Settings_" => "_Enable VLANs_" auf "_Yes_" stellen und die notwendigen Einstellungen vornehmen (Docker- und VM-Dienst muss vorher beendet werden).
- Das neu erstellte VLAN (z.B. "_br0.5_") kann nun bei der Container Erstellung ausgewählt werden.

<br>  

## Vorarbeiten vor der Containererstellung

2. **Verzeichnisse anlegen**  
   Verzeichnis "`docspell`" unter "`appdata`" anlegen.  
   Verzeichnis "`docspell/opt`" unter "`appdata`" anlegen.  

3. **Konfigurationsdatei anlegen**  
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

4. **Gemeinsam genutzte Variablen**  
   Die Docker Container benötigen gemeinsame Variablen (siehe "_docspell.conf_").  
   Die Variablen werden  
   - ENTWEDER einzeln im Telmplate angepasst und verwendet  
     (Die Variablen sind dort bereits angegeben und beschrieben)
   - ODER über eine zentrale Datei unter "_Extra Parameters_" angegeben:  
   "`--env-file=/mnt/user/appdata/docspell/.env`"  
      Es kann die Datei von github genommen werden: <https://github.com/eikek/docspell/blob/master/docker/.env>  
      Sie wird unter "`../appdata/docspell`" gespeichert.  
        
      Die "`.env`"-Datei ist praktisch um Vorgaben zentral zu speichern, erfordert aber das Anlegen der Datei vor Erstellung der Docker  

5. **Unraid Templates einbinden**  
   In Unraid unter "_Docker_" => "_Template Repositories_" (ganz unten) folgende URL eintragen und auf "_Save_" klicken: <https://github.com/vakilando/unraid-docker-templates>  

6. **Container einrichten**  
   In Unraid unter "_Docker_" unten auf "**ADD CONTAINER**" klicken und los geht's  
   <br>
   1. **postgres**   
      Ich verwende Postgresql aus dem original Repository  
      Repository: postgres:11.7  
      Registry: https://hub.docker.com/\_/postgres/  
      Support: https://hub.docker.com/\_/postgres/  

      <br>

      Feld | Wert  (anzupassen!)  
      ---------|----------
       Repository: | postgres:11.7
       Network Type: | custom-network oder VLAN wählen
       Fixed IP address:  | ...sofern gewünscht...
       Extra Parameters |--hostname db
       POSTGRES_PASSWORD | somesafepassword
       POSTGRES_USER | dbuser
       POSTGRES_DB | dbname
       Database Storage Path | /mnt/cache/appdata/postgres/
       Web Interface Port | 5432
      <br>    

   2. **solr**   
      Ich verwende Solr aus dem Repository "_A75G's Repository_"  
      Docker: <https://hub.docker.com/r/bitnami/solr/>  
      Support: https://forums.unraid.net/topic/89502-support-a75g-repo/  
      Project: https://lucene.apache.org/solr/  

      Feld | Wert  (anzupassen!)  
      ---------|----------
       Repository: | bitnami/solr:8
       Network Type: | custom-network oder VLAN wählen
       Fixed IP address:  | ...sofern gewünscht...
       Port: | 8983
       Appdata: | z.B. /mnt/user/appdata/docspell/solr_data
       SOLR_PORT_NUMBER: | 8983
       SOLR_CORE: | my_core
      <br>    
   
   3. **vakilando-docspell-consumedir**  
      
      Feld | Wert (anzupassen!)  
      ---------|----------
       Repository: | eikek0/docspell:consumedir-LATEST
       Network Type: | custom-network oder VLAN wählen
       Fixed IP address:  | ...sofern gewünscht...
       Consumedir: | z.B. /mnt/user/appdata/docspell/docs
       TZ | Europe/Berlin
       DOCSPELL_HEADER_VALUE | SomeRandomString  
       DB_TYPE | postgresql
       DB_HOST | Name des DB-Containers z.B. "db"<br> oder IP-Adresse "192.168.3.2"
       DB_PORT | 5432
       DB_NAME | dbname  
       DB_USER | dbuser  
       DB_PASS | somesafepassword  
      <br>    
   
   4. **vakilando-docspell-joex**  
      
      Feld | Wert 
      ---------|----------
       Repository: | eikek0/docspell:joex-LATEST
       Network Type: | custom-network oder VLAN wählen
       Fixed IP address:  | ...sofern gewünscht...
       Host Port 1: | 7878
       docspell.conf | /mnt/user/appdata/docspell/opt/docspell.conf<br>(_muss vorhanden sein!!_)
       TZ | Europe/Berlin
       DOCSPELL_HEADER_VALUE | SomeRandomString  
       DB_TYPE | postgresql
       DB_HOST | Name des DB-Containers z.B. "db"<br> oder IP-Adresse "192.168.3.2"
       DB_PORT | 5432
       DB_NAME | dbname  
       DB_USER | dbuser  
       DB_PASS | somesafepassword  
      <br>    
   
   5. **vakilando-docspell-restserver**
      Feld | Wert 
      ---------|----------
       Repository: | eikek0/docspell:restserver-LATEST
       Network Type: | custom-network oder VLAN wählen
       Fixed IP address:  | ...sofern gewünscht...
       Host Port 1: | 7880
       docspell.conf | /mnt/user/appdata/docspell/opt/docspell.conf<br>(_muss vorhanden sein!!_)
       TZ | Europe/Berlin
       DOCSPELL_HEADER_VALUE | SomeRandomString  
       DB_TYPE | postgresql
       DB_HOST | Name des DB-Containers z.B. "db"<br> oder IP-Adresse "192.168.3.2"
       DB_PORT | 5432
       DB_NAME | dbname  
       DB_USER | dbuser  
       DB_PASS | somesafepassword  
      <br>    

7. **Die Reihenfolge in der die Container starten ist wichtig!**  
   Daher ist diese nach der Installation durch verschieben mit der Maus ggf. zu korrigieren:  
    1. postgres  
    2. solr  
    3. docspell-consumedir  
    4. docspell-joex  
    5. docspell-restserver  

<br><br>

## Inhalt der .env-Datei  
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

## Genutzte Container Variablen in den Templates

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
Description: | DB_HOST=db<br>database host name  
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
  
  
  



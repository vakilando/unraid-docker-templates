# unraid-docker-templates
### My Unraid Docker User Templates for Docspell  

Attention: this is a personal setup and it will not run "out of the box" on your machine..... 

Docspell:
- https://hub.docker.com/r/eikek0/docspell/  
- https://github.com/eikek/docspell
- https://docspell.org/docs/

be carefull, you MUST change some details to get docspell up and running!
- docker: the templates cannot be used as provided, adjust settings!
- files: the files ".env" and "opt/docspell.conf" cannot be used as provided, adjust settings!
- files: copy the files ".env" and "opt/docspell.conf" to appdata/docspell before starting the containers
- docker: change settings of each app appropriate to your configuration

-----------
-----------


# Installation von Docspell unter Unraid

Um Docspell unter Unraid zu nutzen müssen mehrere Container installiert werden, die miteinander kommunizieren müssen.
Den Containern müssen diverse Variablen mitgegeben werden und eine Konfigurationsdatei.

### Die von mir gewählte Lösung ist:  
In einem neuen Custom Docker Netzwerk werden alle Container erstellt.  
Sie erhalten alle feste IP-Adressen (sowie Mac-Adressen).  
MAC-Adressen-Vergabe ist keine Pflicht, hilft aber ggf. bei Verwendung einer Firewall, wenn nach IP- und/oder MAC-Adressen gefiltern werden soll.  
Momentan werden auch die Container für Postgresql und Solr von mir nur für Docspell genutzt. Natürlich können diese auch für andere Docker/Anwendungen noch zur Verfügung gestellt werden.

### Vorgehen

1. ein eigenes Docspell Docker Netzwerk erstellen:  
   `# docker network create --driver=bridge --subnet=192.168.3.0/25 --gateway=192.168.3.1  custnet-docspell`  

2. Verzeichnis "`docspell`" unter "`appdata`" anlegen  
   Verzeichnis "`docspell/opt`" unter "`appdata`" anlegen  

3. Konfigurationsdatei "`docspell.conf`" in "`/docspell/opt`" anlegen  
   Es kann die originale Datei von github genommen werden (<https://github.com/eikek/docspell/blob/master/docker/docspell.conf>)  
     
   Ich habe folgende Änderungen vorgenommen:  
   - alle Dockernamen durch ihre IP-Adresse ersetzt  
     also z.B. "`http://joex:7878`"  
     => "`http://192.168.3.1:7878`"  
   - alle Variablen durch ihre Werte ersetzt  
     also z.B. "`jdbc:"\${DB_TYPE}"://"\${DB_HOST}":"\${DB_PORT}"/"\${DB_NAME}` => `"jdbc:postgresql://192.168.3.2:5432/dbname"`  
       
   Ich habe es NICHT getestet aber:  
   - durch die Verwendung des custom Docker Netzwerks sollte die Dockernamensauflösung eigentlich funktionieren?!  
   - durch Angabe der Variablen in den Unraid Docker Templates sollten auch die Variablen hier funktionieren?!  

4. Die einzelnen Docker Container benötigen gemeinsame Variablen. Diese werden  
   - ENTWEDER einzeln im Telmplate angepasst und verwendet  
   - ODER als "`--env-file=/mnt/user/appdata/docspell/.env`"  
      unter "_Extra Parameters_" angegeben.  
      Es kann die originale "`.env`"-Datei von github genommen werden (<https://github.com/eikek/docspell/blob/master/docker/.env>)  
      Sie wird unter "`../appdata/docspell`" gespeichert.  
        
      Die "`.env`"-Datei ist praktisch um Vorgaben zentral zu speichern, erfordert aber das Anlegen der Datei vor Erstellung der Docker  

5. Unraid Templates einbinden  
   <https://github.com/vakilando/unraid-docker-templates>  

6. Unten unter "_Docker_" auf "**ADD CONTAINER**" klicken und los geht's  

7. Die Reihenfolge in der die Container starten ist wichtig!  
   Daher diese nach der Installation durch verschieben mit der Maus korrigieren:  
    1. postgres  
    2. docspell-solr  
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

## Container Variablen

_______________________________________________________________  
Variable 1  
Config Type:    Variable  
Name:           TZ  
Key:            TZ  
Value:          Europe/Berlin  
Default Value:  Europe/Berlin  
Description:    TZ=Europe/Berlin  
                Timezone to set  
Display:        Always  
Required:       No  
Password Mask:  No  
  
  
_______________________________________________________________  
Variable 2  
Config Type:    Variable  
Name:           DOCSPELL_HEADER_VALUE  
Key:            DOCSPELL_HEADER_VALUE  
Value:          SomeRandomString  
Default Value:  
Description:    DOCSPELL_HEADER_VALUE=SomeRandomString  
                defines a secret that is shared between some containers. Please see the consumedir.sh docs for additional info.  
Display:        Always  
Required:       Yes  
Password Mask:  No  
  
_______________________________________________________________  
Variable 3  
Config Type:    Variable  
Name:           DB_TYPE  
Key:            DB_TYPE  
Value:          postgresql  
Default Value:  postgresql  
Description:    DB_TYPE=postgresql  
                type of database to use (postgresql, mariadb, h2)  
                You need an allready running database server  
Display:        Always  
Required:       Yes  
Password Mask:  No  
  
_______________________________________________________________  
Variable 4  
Config Type:    Variable  
Name:           DB_HOST  
Key:            DB_HOST  
Value:          db  
Default Value:  db  
Description:    DB_HOST=db  
                database host name  
Display:        Always  
Required:       Yes  
Password Mask:  No  
  
_______________________________________________________________  
Variable 5  
Config Type:    Variable  
Name:           DB_PORT  
Key:            DB_PORT  
Value:          5432  
Default Value:  5432  
Description:    DB_PORT=5432  (postgresql)  
                database host port  
Display:        Always  
Required:       Yes  
Password Mask:  No  
_______________________________________________________________  
Variable 6  
Config Type:    Variable  
Name:           DB_NAME  
Key:            DB_NAME  
Value:          dbname  
Default Value:  dbname  
Description:    DB_NAME=dbname  
                database name  
                database will be created at first run  
Display:        Always  
Required:       Yes  
Password Mask:  No  
_______________________________________________________________  
Variable 7  
Config Type:    Variable  
Name:           DB_USER  
Key:            DB_USER  
Value:          dbuser  
Default Value:  dbuser  
Description:    DB_USER=dbuser  
                database user name  
Display:        Always  
Required:       Yes  
Password Mask:  No  
_______________________________________________________________  
Variable 8  
Config Type:    Variable  
Name:           DB_PASS  
Key:            DB_PASS  
Value:          somesafepassword  
Default Value:  somesafepassword  
Description:    DB_PASS=somesafepassword  
                database password  
Display:        Always  
Required:       Yes  
Password Mask:  No  
  
  
  



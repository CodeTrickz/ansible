# Projectopdracht

Welkom bij het de repository voor de doorlopende opdracht van DevOps. 

## Situatie

Je werkt voor ACME. Ze hebben onlangs een todo-applicatie ontwikkeld. 
Je wordt gevraagd om een DevOps pipeline uit te werken. Hiervoor zal je op een remote Linux machine werken.

Er zijn twee onderdelen: Back-end en Front-end. Er is geen authenticatie.

De back-end is een NodeJs applicatie die de API host. De back-end luistert op poort 3000. 
De back-end draait standaard in-memory. Met andere woorden, de taken worden enkel opgeslagen in geheugen, niet op disk. 
De back-end kan ook een connectie maken naar een mysql databank. Die stel je in met volgende environment variabelen:

* STORAGE=mysql
* MYSQL_HOST=<hostname>
* MYSQL_USER=<username>
* MYSQL_PWD=$mysqlpwd 
* MYSQL_DB=$mysqldb

De Front-end is een HTML5 statische applicatie en luistert op poort 80. 
Via AJAX calls wordt de API aangeroepen. De front-end en de API moeten op dezelfde host draaien. 
De API draait in een subfolder /api.

## Architectuur

![Architectuur](./architectuur.png)

## Opdracht:

1. Connecteer naar je Debian Linux omgeving
1. Clone deze repository
1. Bouw een container voor de back-end. De back-end draait op NodeJS. Kies zelf een tag.
1. Bouw een container voor de front-end. De front-end draait op Nginx. Je maakt gebruik van nginx. Je kan gebruik maken van de standaard configuratie.
1. Maak een docker compose file aan die deze containers refereert. Expose je front-end op poort 80.
1. Voeg Mysql toe aan je netwerk dmv een docker container. Je maakt gebruik van een mysql image. Voeg disk mappings toe zodat de state van je mysql container bewaard blijft.
1. De repository bevat een bestand init.sql. Zorg dat dit wordt uitgevoerd bij de start van de Mysql container. 
1. Configureer de API zodat die deze MySql databank gebruikt.
1. Installeer Traefik als Reverse proxy op je omgeving.
1. Configureer ssl certificaat aanvraag via LetsEncrypt
1. Expose de todo applicatie op ```https://<studentnr>.devops-ap.be```

## werking
### doel
Met alle toevoegingen tracht ik om alle extra containers die bovenop de base app draaien, Op één host te laten werken. In realiteit ga je ook vaak maar één domein hebben en niet meerdere. Vanuit dat standpunt wil ik alles op één domein. 

### toevoegingen
#### access logs
Om de access logs te kunnen wegschrijven moet er map gemaakt worden met de naam logs met daarin access.json. De access logs zullen daarin worden weggeschreven. Vanuit security overwegingen staat de logs map standaard in de .gitignore

#### jaeger tracing
Voor dit project wordt traefik gebruikt als reverse proxy. De jaeger tracing is een extra feature dat bovenop trafik draait voor tracing. De jaeger UI draait op "https://s141086-1.devops-ap.be/jaeger". Dankzij de environment variabele "- QUERY_BASE_PATH=/jaeger" in traefik docker compose , werkt jaeger op /jaeger en niet op de root van de host.

#### traefik
Traefik wordt in dit project als reverse proxy voor alle onderliggende containers. De UI is beschikbaar op "https://s141086-1.devops-ap.be/". De login van de user wordt gehashed bijgehouden in de traefik map als passwd. Deze map moet je aanmaken. 

Voor het certificaat moet je een bestand acme.json aanmaken. De mail in de traefik.yml moet je aanpassen naar je eigen email. Letsencrypt doet de rest. 

#### jenkins
De Jenkinscontainer wordt gebruikt om de base app te builden en deployen. Doormiddel van agent en een pipeline script. Je moet hier inloggen met je de user die je aanmaakt tijdens de jenkins installatie. De jenkins UI draait op "https://s141086-1.devops-ap.be/jenkins". Dankzij de environment variabele "- JENKINS_OPTS=--prefix=/jenkins" in docker compose van jenkins , werkt jenkins op /jenkins en niet de root van de host.

##### automatisch build met jenkins
Default word er met elke git push , de applicatie automatisch opnieuw gebuild en gedeployed met jenkins. Dit gebeurd via een webhook die aangemaakt is in het github project. Deze webhook is gekoppeld aan de jenkins agent. 
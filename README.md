# docker

Docker:
Docker ist eine Art der Virtualisierung.

Vorteile:
-> Isolierung von Systemen für mehr sicherheit.
-> Der Container kann auf aller Arten von Umgebungen ausgeführt werden.

Container (docker) vs. Virtuelle Machinene
-> Virtuelle Machinene: HW -> OS -> Hypervisor -> Gast-OS -> Anwendung

-> Container: HW -> OS -> Container-Engine -> Anwendung

Um einen Container zu bauen, kann direct ohne das Gast-OS auf das Container-Engine gebaut werden. Problem ist Daten-persistence. Das Neustarten ist wie Reset. Daten werden gelöscht.

Container Für Cloud and DevOps
VM für Betriebsysteme zum Testen zum Cyber-Security

Images vs Container:
Docker arbeitet mit Images und Container. Images können als eine Art Template gesehen werden. Der Container ist eine Instanz eines Templates.

Wenn man es mit objekt-orientierter Programmierung vergleichen will, ist ein Image eine Klasse und ein Container eine Instanz.


Mit docker einen Container erstellen
>docker pull busybox // lande das image busybox von Docker site herunter.
>docker images // zeige das installierte docker images 
>docker run busybox // starte den docker busybox
>docker ps // zeige Containers, die am Laufen sind.
>docker ps -a // zeige ALLE Containers, die am Laufen sind.
>docker rm $container_id // container löschen.

>docker run -it busybox sh // starte den Container mit shell. -it for interactive mode 

## Container starten
>docker pull nginx
>docker run nginx 
jetzt, wie greifen wir auf den nginx server zu? Docker erlaubt kein Zugriff von draußen, deswegen sollen wir ein port forwarding in unserem System machen. localhost port soll auf den docker container geleitet wird.
>docker run -p local_port:docker_port -d nginx
>docker run -p 5000:80 -d nignex
5000 auf localhost leitet weiter auf 80 port auf den docker. -d steht für detached mode.

## Container/Images stoppen/löschen
>docker run -p 5000:80 -d nginx // docker starten
>docker ps // zeige Container, die am Laufen sind
>docker stop $container_id
>docker rm $container_id

>docker images ls // zeige die Images
>docker ps -a 
>docker container prune
>docker image rm nginx

>docker exec -ti $container_hash bash // offene bash shell on the container
>docker attach $container_hash // attach mode 

## Ordner verknüpfen/verlinken mit -v
>docker run -p 5000:80 -d  -v local_ordner:server_ordner nginx

>systemctl start/restart/stop docker

>docker stats // Statistiken der laufenden Container

## Die Ressourcen eines Containers limitieren mit -m and --cpus
>docker run -p 5000:80 -d  -m 4m/4g/4t  --cpus="Anzahl der Kerne" nginx

## Container automatisch starten lassen mit
--restart no/always/unless-stopped/on-failure
>docker run -p 5000:80 -dit --restart unless-stopped nginx 


## Logs ansehen & aktivieren
>docker logs $container_id
>docker run -p 5000:80 --log-driver json-file --log-opt max-size=100m --log-opt max-file=100 nginx
>docker inspect -f {{.LogPath}} $container_id


## Dockerfile
Im Dockerfile steht wie der Container gebaut werden soll.
```
FROM python:3.7-slim

# working directory, hier will ich alles liegen 
WORKDIR /app 

# Kopieren des aktuellen Verzeichnisses in /app
ADD . /app

# braucht man nicht (optional) aber schadet auch nicht
RUN pip install --trusted-host pypi.python.org -r requirements.txt 

# Port 80 veröffentlichen
EXPOSE 80

## Environment Variablen
ENV NAME Ibrahim


CMD ["python","app-py"]
```
>docker build -t erstesProjekt $DockerFileOrdner
>docker images 
>docker run -dit -p 80:80 erstesProjekt
>docker stop $container_id

Wenn wir was ändern, muss wir den Container neubilden. 


## Images auf Docker Hub teilen
Anmeldung unter hub.docker.com ist erforderlich
>docker tag erstesProjekt $Benutzername/projektName:1.0
>docker login
>docker push $Benutzername/projektName:1.0
>docker pull $Benutzername/projektName
>docker run $Benutzername/projektName:1.0


## Environment Variablen
- Für das Speichern von Sensible Inforamtionen
ENV NAME Ibrahim
>docker run -it -p 5000:80 -e NAME=Ibahim erstesProjekt // hier wird den Variablen nur für diesen Container gilten


## Ein Email Server mit MailCow
clone mailcow-dockerized von Github 
führe den generate-config.sh aus
>docker-compose pull
>docker-compose up -d


## Wordpress mit Docker installieren
```
version: "3.6" // version der docker-compose

services:
	db:
		image: mysql:5.7 
		volumes:
			- data_db:/var/lib/mysql
		ports:
			- 3306:3306
		restart: always
		environment:
			MYSQL_ROOT_PASSWORD: mypassword
			MYSQL_DATABASE: wordpress
			MYSQL_USER: wordpress
			MYSQL_PASSWORD: wordpress
		
		wordpress:
			image: wordpress:latest
			depends_on:
				- db
			ports:
				- 80:80
			restart: always
			environment:
				WORDPRESS_DB_HOST: db:3306
				WORDPRESS_DB_USER: wordpress
				WORDPRESS_DB_PASSWORD: wordpress
			deploy:
				replicas: 3 // wie oft soll der Container gestartet werden soll
				resources: // limitierung von arbeitsspeicher oder CPU und so weiter
					- cpus: "0.5"
					memory: 500m
			volumes:
				- wordpress/plugins:/var/www/html/wp-content/plugins
volumes:
	data_db:
```
>docker-compose up -d
>docker swarm init; docker stack deploy -c docker-compose.yml wordpress

docker hat nichts mit docker-compose was zutun, beide sind unabhängige Anwendung

## Eine eigene Cloud mit Nextcloud
>docker pull nextcloud
>docker run -dit -p 80:80 nextcloud


# Swarm
ähnlich wie loadbalancing. Also mehrere Nodes werden zu einem Cluster (Swarm) zusammengefasst.

# Docker Machine auf Linux installieren
>pacman -S docker-machine
>base=https://github.com/docker/machine/releases/download/v0.16.0
>curl $base/docker-machine-$(uname -s) -$(uname -m) > /tmp/docker-machine
>install /tmp/docker-machine /user/local/bin/docker-machine
>docker-machine version


## Lokale virtuelle Maschinen erstellen
>docker-machine create --driver virtualbox myvm
>docker-machine ls // zeige die installierte maschinen
>docker-machine stop myvm
>docker-machine rm myvm
>docker-machine inspect myvm // zeige infos in details über eine bestimmte vm

## Einen Swarm erstellen
>docker-machine ls // zeige die gestellte VM
>docker swarm leave --force // swarm verlassen, falls du in einem swarm seid
>docker swarm init --advertise-addr 192.168.178.111 // erstelle einen swarm mit dem ip 192.168.178.111
>docker-machine ssh myvm "docker swarm join ......" // füge meine VM in dem Swarm hinzu
>docker node ls // zeige meine Nodes
>docker-compose up -d
>docker-compose down
>docker stack deploy -c docker-compose.yml wordpress
>docker stack ps

## Den Stack beenden, den Swarm verlassen
>docker stack rm wordpress
>docker-machine ssh myvm "docker swarm leave" // beende eine Machine
>docker swarm leave -f
>docker-machine ls
>docker-machine stop $(sudo docker-machine ls -q) // beende alle Machinen
>docker-machine rm $(sudo docker-machine ls -q) // beende alle Machinen



Befehle:
> sudo docker info // get docker infos
-> docker build -t „imagename“
buildet ein Docker Image aus einer Datei namens „Dockerfile“ (ohne Dateiendung), das Image bekommt den Namen der mit `-t` als Parameter übergeben wurde.


-> docker run -d -p 8080:80 „imagename“  
erstellt und startet einen Container basierend auf den Images,
"-p"  mappt den port 8080 (des Hostsystems) im Container auf den port 80.
"-d"  startet den Container im detached mode, der Container wird im Hintergrund gestartet und die console wird nicht von der Ausgabe des Docker Containers blockiert.

-> docker images  
zeigt alle gebauten oder heruntergeladenen Images an

-> docker image rm "imagename"  
entfernt das angegebene Image

-> docker container ls -a  
zeigt alle laufenden Container an

-> docker start "CONTAINER_Name"  
startet den angegeben Container

-> docker logs -f (CONTAINER | SERVICENAME)
zeigt den log output des Containers

-> docker stop/kill/rm CONTAINER
stop: stoppt den angegebenen Container
kill: wird mit einem kill signal gestoppt
rm: entfernt den Docker Container (nur den Container nicht das Image)



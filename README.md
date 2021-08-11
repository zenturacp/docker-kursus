# Mini Docker Kursus
Hvad er en Container?

Lidt som i gamle dage med chroot hvor en process tror den har rod i en anden mappe, forskellen er bare at Docker er på kerne niveau og bruger cgroups (lang forklaring - der ikke kommer her)

Se docker som en "orkerstrerings motor" ikke som en "app" der køre, men en samling af værktøjer og et API til dette.

Den primære driver er den Overlay Driver docker har udviklet hvor man kan "merge" lag i en container

![1-etcher-3.png](https://github.com/zenturacp/docker-kursus/raw/main/images/docker_image_explained.png)

## Fordele ved Docker
* Kode og Applikation hænger sammen - det er det største Win
* Mindre komponenter (Mikroservices) - Alpine Linux (5mb Linux)
* Du kan køre med minimal applikationer = sikkerhed, for der er færre komponenter du skal vedligeholde
* Du kan automatisere CI/CD - Dev / Test / Pre Prod / Prod - alt lavet med samme installation
* Du kan skalere med f.eks. Kubernetes

## Hvordan kan man få Docker
* Som lokal installation på en Linux
* Via Docker Desktop (Deres udviklings værktøj)

## Installation
Fra din favorit Linux installation køre du følgende

```
curl https://get.docker.com | sudo bash
```

Din bruger skal være medlem af Docker gruppen for at kunne bruge docker

```
sudo usermod -a -G docker om
```

For at installere Docker Compose kør følgende kommando
```
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

Alternativ kan man installere docker compose via pip (F.eks. på RaspBerry) dette kræver Python er installeret på maskinen

```
sudo pip install docker-compose
```

## Køre Hello World
```
docker run --rm hello-world
```

## Docker forklaret

Docker består af flere komponenter

### Container

Metadata + et Write Layer

### Image

Read Only filer som kan bruges startes via en Container

### Volume

Et virtuel filsystem som kan bruges i container til at gemme data

Du kan også "mappe" en mappe / fil ind i en container istedet for at bruge volumes

### Netværk

Hvordan skal man kunne tilgå sin container - [Online Dokumentation](https://docs.docker.com/network/)

* Bridge
  * Default mode, din container vil få en IP på et lukket netværk, og du skal definere hvad porte der skal ind til container
* Host
  * Din Container kommer direkte på din Host (PC dvs. ikke isoleret og alle porte i Container vil være tilgængelige)
* Overlay
  * Ikke omfattet her men bruges hvis du har flere docker servere i et Cluster
* Service Discovery
  * Docker sørger for at alle containers har et DNS navn på de netværk som bliver oprettet, dvs. på Docker bruger man aldrig IP adresser

## Eksemple på en applikation (Den trivielle måde)

I dette eksempel laver vi et WordPress site med MySQL

Først laver vi et Netværk (Dette skal vi gøre for containere kan snakke sammen)
```
docker network create backend
```

Lave en MariaDB Container på din maskine

```
docker run \
    --name mariadb-server \
    --network backend \
    -e MARIADB_ROOT_PASSWORD=OrangeMakers2021 \
    --detach \
    mariadb
```

Nu skal vi lave en Database, WordPress User til WordPress, samt en Admin bruger til senere brug

For at få adgang til SQL Serveren kan man inde fra containeren mariadb køre mariadb (Klienten) dette gøres med følgende kommando

```
docker exec -it mariadb-server mariadb -p
```

Ovenstående kommando vil køre filen mariadb i containeren der hedder mariadb-server, med parametret -p efter (Spørg for password), -it siger blot at du skal kunne sende keyboard strokes til den og den først skal stoppe når filen lukker igen

```sql
CREATE DATABASE wpdb;
CREATE USER 'wpuser'@'%' IDENTIFIED BY 'OrangeMakers2021';
GRANT ALL ON wpdb.* to 'wpuser'@'%' IDENTIFIED BY 'OrangeMakers2021' WITH GRANT OPTION;
CREATE USER 'admin'@'%' IDENTIFIED BY 'OrangeMakers2021';
GRANT ALL PRIVILEGES ON *.* to 'admin'@'%' IDENTIFIED BY 'OrangeMakers2021' WITH GRANT OPTION;
FLUSH PRIVILEGES;
EXIT;
```

Lav en WordPress Container på din maskine

```
docker run \
    --name wordpress \
    --network backend \
    --detach \
    -e WORDPRESS_DB_HOST=mariadb-server \
    -e WORDPRESS_DB_USER=wpuser \
    -e WORDPRESS_DB_PASSWORD=OrangeMakers2021 \
    -e WORDPRESS_DB_NAME=wpdb \
    -p 80:80 \
    wordpress
```

For at administrere vores MariaDB er det rart med PHP MyAdmin installeret dette kan gøres på følgende måde

```
docker run \
    --name myadmin \
    --network backend \
    -e PMA_HOST=mariadb-server \
    -p 8080:80 \
    phpmyadmin
```

## Samme eksempel som ovenfor men nu med Docker-Compose

Docker Compose er en "deklarativ" måde at sætte dit docker miljø op på, docker compose kigger på din template og sørger for det er som der står i templaten. dvs. man kan køre denne igen og igen og rette ting i den som så vil kunne lægges på.

```yaml
version: "3"
services:

  mariadb-server:
    image: mariadb:latest
    container_name: mariadb-server
    restart: always
    networks:
      - backend
    environment:
      - MYSQL_ROOT_PASSWORD=OrangeMakers2021

  wordpress:
    image: wordpress:latest
    container_name: wordpress
    restart: always
    networks:
      - backend
    ports:
      - "80:80"
    environment:
      - WORDPRESS_DB_HOST=mariadb-server
      - WORDPRESS_DB_USER=wpuser
      - WORDPRESS_DB_PASSWORD=OrangeMakers2021
      - WORDPRESS_DB_NAME=wpdb

  myadmin:
    image: phpmyadmin:latest
    container_name: myadmin
    restart: always
    networks:
      - backend
    ports:
      - "8080:80"
    environment:
      - PMA_HOST=mariadb-server
    depends_on:
      - "mariadb-server"

networks:
  backend:
```

## Docker Hub
Hvad er docker Hub?

## Bygge Docker image
Et Docker image består af en Dockerfile som er en fil der bestemmer hvordan din docker skal bygges. jeg har lavet et eksemple her

Dette er blot en Webserver baseret på NGINX med en Index fil

Her et absolut minimum [eksempel](https://github.com/zenturacp/docker-kursus/tree/main/buildwebserver-static)

```dockerfile
FROM nginx:latest

COPY index.html /usr/share/nginx/html/
COPY phpinfo.php /usr/share/nginx/html/
```

Hvis man skal bruge PHP findes der en PHP Container fra PHP selv

```dockerfile
FROM nginx:latest

COPY index.html /usr/share/nginx/html/
COPY phpinfo.php /usr/share/nginx/html/
```

## Cheat Sheet
* Start Container
  * docker run container-navn
* Stop Container
  * docker stop container-navn
* List kørende containere
  * docker ps
* List alle også stoppede containere
  * docker ps -a
* Se logs fra Container
  * docker logs container-navn
* Gå ind i Container
  * docker exec -it container-navn (bash eller sh)

## Maskiner til kursus

| Hostname | IP adresse    | Username | Password         |
| -------- | ------------- | -------- | ---------------- |
| DOCKER1  | 20.101.95.199 | om       | OrangeMakers2021 |
| DOCKER2  | 13.80.171.197 | om       | OrangeMakers2021 |
| DOCKER3  | 13.80.171.208 | om       | OrangeMakers2021 |
| DOCKER4  | 13.80.171.248 | om       | OrangeMakers2021 |

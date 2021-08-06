# Mini Docker Kursus

Hvad er en Container?

Lidt som i gamle dage med chroot hvor en process tror den har rod i en anden mappe, forskellen er bare at Docker er på kerne niveau og bruger cgroups (lang forklaring - der ikke kommer her)

Se docker som en "orkerstrerings motor" ikke som en "app" der køre, men en samling af værktøjer og et API til dette.

Den primære driver er den Overlay Driver docker har udviklet hvor man kan "merge" lag i en container

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

## Køre Hello World

```
docker run --rm hello-world
```

## Docker Elementer

### Container

### Image

### Volume

### Netværk

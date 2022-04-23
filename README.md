# mini-projet-docker Version 2
Déploiement d'un micro-service flask python

**Objectif du projet :** Valider les acquis sur le module **Docker**.

Dans ce mini-projet-docker, il est question pour nous d'embarquer une application dans un container docker. Cette application se trouve dans repository git du lien suivant [dirane](https://github.com/diranetafen/student-list)

# Résolution explicative du projet

## Partie 1: Build et test de l'application
Pour ce faire:
- Nous allons mettre en place un **Dockerfile** selon les instructions données dans l'énoncé.
- Cette application sera ensuite tester via un curl.

## Partie 2: Mise en place de l'Infra As Code (IAC)
Pour ce faire:
- Nous allons utiliser l'outil **docker-compose**. Cet outil va nous permettre d'automatiser le déploiement de l'application que nous allons conteneuriser.
- Nous allons aussi rajouter l'IHM avec un container provenant de l'image **php:apache**. Cette **IHM** va nous permettre de tester le fonctionnel de notre application via un navigateur.

En résumé, notre **docker-compose** devra nous déployer deux containers qui sont:
- Container 1: notre application (api développée en flask python)
- Container 2: L'**IHM** qui part taper sur le **container 1** afin de nous donner la possibilité de tester le fonctionnel de notre application.

## Partie 3: Docker Registry

Une fois que nous aurons notre application embarquée dans le container docker, on va créer l'image. Cette dernière est en locale.

Il sera donc question d'enterriner cette image dans un **registry**. Pas le **dockerhub** car il est public et nous avons juste droit droit à un seul compte privé et un gratuit.

Nous allons donc déployer un troisième container qui aura pour rôe d'être le **registry** où nous pourrons envoyer notre image nouvellement buildée dans ce **Registre** local.

Pour communiquer, nos quatre conainer doivent être dans un réseau de type **Bridge**. Ce dernier nous permet d'exposer nos application **IHM** et **IHM Registry** à l'extérieur. Il faudra mettre les règles d'exposition des ports.

Seul les containers **IHM** seront exposés. Par exemple le container **IHM** va écouter via le port 80 et **IHM Registry** via le port 8082

**Fin du mini-projet-docker**.

# Illustaration schématique de la solution

![Image Solution](/images/illust.png "Image Solution Projet")

# Réalisation
# Partie 1: Build et test
### étape 1: Nous allons déployer la machine qui contient docker
Pour ce déploiement, je vais utiliser:
- L'hyperviseur de type 2 **VirtualBox** que j'installe.
- Le script d'installation de **docker**, que je démarre avec *start*, je le rends disponible avec *enable* et j'ajoute l'utilisateur *vagrant* avec ses droits afin de ne plus taper *sudo* poue exécuter les commande.
- Le provider **vagrant** qui va automatiser le déploiement de la VM et l'installation de docker.

Pour créer la VM docker dans VirtualBox, je tape la commande suivante:
```
vagrant up --provision
```
Une fois ma VM mise en place, je me connecte à celle-ci via **ssh** (il va utiliser une clé privée qu'il a eu à générer) par la commande:
```
vagrant ssh
```
Une fois la connexio établie, nous avons le rendu ci-dessous. Alors nous sommes déjà connectés à notre VM.
```
[vagrant@docker ~]$
```
Je vais vérifier que ma VM est véritablement déployée.
```
[vagrant@docker ~]$ uptime
```
Résultat:
```
06:23:16 up 20 min,  1 user,  load average: 0.00, 0.05, 0.16
```
Je vérifie l'OS .
```
[vagrant@docker ~]$ cat /etc/redhat-release 
CentOS Linux release 7.9.2009 (Core)
```
Je vérifie si j'ai une ou des images qui tourne(nt) dans la VM.
```
[vagrant@docker ~]$ docker images
```
Je vérifie si j'ai le ou les container(s) qui tourne(nt)
```
[vagrant@docker ~]$ docker ps -a
```
### étape 2: Phase du build
- Je télécharge le repository git du projet. Pour ce faire, j'installe d'abord *git* dans ma VM s'il n'est pas encore.
```
[vagrant@docker ~]$ sudo yum install git -y
```
- Puis je clone le projet qui est dans le repository *git*.
```
[vagrant@docker ~]$ git clone https://github.com/diranetafen/student-list.git
```
Clone réussi et nous avons le rendu ci-dessous.
```
Cloning into 'student-list'...
remote: Enumerating objects: 27, done.
remote: Counting objects: 100% (10/10), done.
remote: Compressing objects: 100% (7/7), done.
remote: Total 27 (delta 4), reused 3 (delta 3), pack-reused 17
Unpacking objects: 100% (27/27), done.
```
Je liste tout ce qui se trouve dans le repertoire
```
[vagrant@docker ~]$ ls
```
Résultat:
```
get-docker.sh  student-list
```
Je liste tout ce qui se trouve dans le repertoire avec un plus de détail et ressortir les dossiers et fichiers cachés.
```
[vagrant@docker ~]$ ls -al
```
```
total 36
drwx------. 6 vagrant vagrant   164 Apr 22 06:34 .
drwxr-xr-x. 3 root    root       21 Feb 12 17:49 ..
drwx------. 3 vagrant vagrant    37 Feb 12 18:13 .ansible
-rw-r--r--. 1 vagrant vagrant    18 Apr  1  2020 .bash_logout
-rw-r--r--. 1 vagrant vagrant   193 Apr  1  2020 .bash_profile
-rw-r--r--. 1 vagrant vagrant   231 Apr  1  2020 .bashrc
-rw-r--r--. 1 root    root    18617 Apr 22 06:07 get-docker.sh
drwxrw----. 3 vagrant vagrant    19 Apr 22 06:34 .pki
drwx------. 2 vagrant vagrant    29 Apr 22 06:02 .ssh
drwxrwxr-x. 5 vagrant vagrant    94 Apr 22 06:34 student-list
-rw-r--r--. 1 vagrant vagrant     6 Feb 12 17:58 .vbox_version
```
Je liste juste le contenu du repertoire cloné.
```
[vagrant@docker ~]$ ll
```
Résultat: Un fichier et un dossier.
```
total 20
-rw-r--r--. 1 root    root    18617 Apr  6 17:38 get-docker.sh
drwxrwxr-x. 5 vagrant vagrant    94 Apr  6 20:22 student-list 
```
Je me positionne dans le repertoire de travail (où se trouve l'application à builder)
```
[vagrant@docker ~]$ cd student-list/
```
Résultat:
```
[vagrant@docker student-list]$
```
Je liste juste le contenu du repertoire de travail.
```
[vagrant@docker student-list]$ ll
```
Résultat:
```
total 8
-rw-rw-r--. 1 vagrant vagrant    0 Apr 22 06:34 docker-compose.yml
-rw-rw-r--. 1 vagrant vagrant 7133 Apr 22 06:34 README.md
drwxrwxr-x. 2 vagrant vagrant   70 Apr 22 06:34 simple_api
drwxrwxr-x. 2 vagrant vagrant   23 Apr 22 06:34 website
```

Pour avoir **l'image**, il me faut un **Dockerfile** que je vais builder.

Le **Dockerfile** qui se trouve dans **simple_api**, je me positionne dans ce repertoire.
```
[vagrant@docker student-list]$ cd simple_api/
```
Résultat:
```
[vagrant@docker simple_api]$
```
Je liste juste le contenu de mon repertoire de travail.
```
[vagrant@docker simple_api]$ ll
```
Résultat:
```
total 8
-rw-rw-r--. 1 vagrant vagrant    0 Apr 22 06:34 Dockerfile
-rw-rw-r--. 1 vagrant vagrant   39 Apr 22 06:34 student_age.json
-rwxrwxr-x. 1 vagrant vagrant 1667 Apr 22 06:34 student_age.py
```
J'ouvre le **Dockerfile** avec l'éditeur de test *vi*.
```
[vagrant@docker simple_api]$ vi Dockerfile
```
Puis je vais renseigner ce **Dockerfile** des consignes de l'énoncé
```
FROM python:2.7-stretch
MAINTAINER Daniel MEDOU
ADD student_age.py /
RUN apt-get update -y && apt-get install python-dev python3-dev libsasl2-dev python-dev libldap2-dev libssl-dev -y
RUN pip install flask==1.1.2 flask_httpauth==4.1.0 flask_simpleldap python-dotenv==0.14.0
VOLUME [ "/data" ]
EXPOSE 5000
CMD [ "python", "./student_age.py" ]
```
Mon **Dockerfile** est prêt à être lancé pour construire mon *image*.

Maintenant, je vais passer à la phase de **build**.

Pour ce faire, je suis déjà positionner dans le repètoire où se trouve mon **Dockerfile**, je tape la commande ci-dessous:
```
[vagrant@docker simple_api]$ docker build -t daniel-pozos:V1 .
```
Résultat: Le build s'est passé avec succès
```
Successfully built 9316331a49a5
Successfully tagged daniel-pozos:V1
```
Je vérifie que mon image a été créé et est bien présente.
```
[vagrant@docker simple_api]$ docker images
```
```
REPOSITORY     TAG           IMAGE ID       CREATED         SIZE
daniel-pozos   V1            9316331a49a5   2 minutes ago   1.13GB
python         2.7-stretch   e71fc5c0fcb1   2 years ago     928MB
```
Mon image est bien présente avec le tag V1. Elle est partie d'une image python 2.7 qui a été téléchargée depuis le *Registry Officiel* de docker.

Mon image étant présente, il me reste juste à la tester pour voir ce qu'elle contient.

### étape 3: Phase de test

Pour tester mon *image*, je vais me créer un réseau. Sans option particulière donnée, c'est un réseau de type **Bridge** qui est créé. Je peux aussi mettre l'option **--driver=bridge** dans la commande de création du réseau pour plus de précision du type de réseau que nous utilisons.
```
[vagrant@docker simple_api]$ docker network create daniel-pozos_network 
```
ou utiliser la commande ci-dessous en indiquant le type de **driver** pour bien spécifier le type de réseau.
```
docker network create --driver <DRIVER TYPE> <NETWORK NAME>
```
```
docker network create --driver bridge daniel-pozos_network
```
Je vérifie que mon réseau a été créé en listant. Mon réseau créé est bien présent.
```
[vagrant@docker simple_api]$ docker network ls
```
```
NETWORK ID     NAME                   DRIVER    SCOPE
a51e9f3ce517   bridge                 bridge    local
faf514961911   daniel-pozos_network   bridge    local
32c790d7560b   host                   host      local
02b11bc7611e   none                   null      locall
```
Il est possible de récolter des informations sur le réseau docker, comme par exemple la config réseau, en tapant la commande suivante :
```
docker network inspect daniel-pozos_network
```
Résultat:
```
[
    {
        "Name": "daniel-pozos_network",
        "Id": "c36e67be0cc2ce754311ca1bb01e427131168b7522658e8fb2a26ffb7a96da38",
        "Created": "2022-04-23T13:34:36.062620009Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.19.0.0/16",
                    "Gateway": "172.19.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]
```
Je vais lancer (déployer) mon premier container dans ce réseau **daniel-pozos_network**

```
[vagrant@docker simple_api]$ docker run -d --network daniel-pozos_network --name daniel-test_api_pozos -v ${PWD}/student_age.json:/data/student_age.json -p 4000:5000 daniel-pozos:V1
```
ou
```
[vagrant@docker simple_api]$ docker run -d --network daniel-pozos_network --name daniel-test_api_pozos -v $PWD/student_age.json:/data/student_age.json -p 4000:5000 daniel-pozos:V1
```
Résultat:
```
b71d1d95885f1f2f8a018a1a0b42e63df72500aed9a0ccefb18b6d3697f639b8
```

Je vérifie que mon container a bien été déployé en listant ces derniers.
```
[vagrant@docker simple_api]$ docker ps -a
```
Résultat: j'ai bien mon container 
```
CONTAINER ID   IMAGE             COMMAND                  CREATED          STATUS          PORTS                                       NAMES
b71d1d95885f   daniel-pozos:V1   "python ./student_ag…"   30 seconds ago   Up 29 seconds   0.0.0.0:4000->5000/tcp, :::4000->5000/tcp   daniel-test_api_pozos
```

Dans l'énoncé, nous avons la commande qui va nous permettre de tester l'image. Veuillez la retrouver ci-dessous.

```
curl -u toto:python -X GET http://<host IP>:<API exposed port>/pozos/api/v1.0/get_student_ages
```

Etant en local, je vais tester avec l'ip 127.0.0.1 et le port d'écoute 4000
```
curl -u toto:python -X GET http://127.0.0.1:4000/pozos/api/v1.0/get_student_ages
```
C'est ok. J'ai bien mes données

```
{
  "student_ages": {
    "alice": "12",
    "bob": "13"
  }
}
```

ou 

```
curl -u toto:python -X GET http://127.0.0.1:4000/pozos/api/v1.0/get_student_ages -I
```
Résultat: HTTP/1.0 200 OK nous permet de confirmer que tout se passe bien.
```
HTTP/1.0 200 OK
Content-Type: application/json
Content-Length: 64
Server: Werkzeug/1.0.1 Python/2.7.18
Date: Fri, 22 Apr 2022 07:38:57 GMT
```

**Fin de la phase de build et du test**

# Partie 2: Infrastructure As Code (IAC)
L'objectif de cette partie est de mettre en place un **docker-compose**. Ce dernier nous permettra d'automatiser le déploiement de notre infrastructure.

Dans le problème à résoudre, nous pouvons lancer nos images en deux fois et avec les commances ci-dessous:
```
docker run -d --network daniel-pozos_network --name daniel-test_api_pozos -v $PWD/student_age.json:/data/student_age.json -p 4000:5000 daniel-pozos:V1
```
et

```
docker run -d --network daniel-pozos_network --name Website-daniel_pozos -v $PWD/website:/var/www/html -p 80:80 php:apache
```

Nous allons nous servir de ces deux commandes pour mettre en place le **docker-compose.yml** afin d'automatiser l'installation des containers à partir de chaque image. 

Je présente pour chaque commande un *mini docker-compose.yml*.

Pour procéder à cette étape, nous allons déjà nous positionner dans le répertoire de travail où se trouve le **docker-compose.yml**
```
[vagrant@docker simple_api]$ cd ..
```
```
[vagrant@docker student-list]$ ll
total 8
-rw-rw-r--. 1 vagrant vagrant    0 Apr 22 06:34 docker-compose.yml
-rw-rw-r--. 1 vagrant vagrant 7133 Apr 22 06:34 README.md
drwxrwxr-x. 2 vagrant vagrant   70 Apr 22 07:07 simple_api
drwxrwxr-x. 2 vagrant vagrant   23 Apr 22 06:34 website
```

Ce [lien](https://www.composerize.com/) est un outil en ligne nous permettant de créer facilement et rapidement le *docker-compse* à partir des commandes *docker run* bien montées.

- Cas 1:
```
docker run -d --network daniel-pozos_network --name daniel-test_api_pozos -v $PWD/student_age.json:/data/student_age.json -p 4000:5000 daniel-pozos:V1
```
**docker-compose.yml** du **cas 1:
```
version: '3.3'
services:
    daniel-pozos:
        network_mode: daniel-pozos_network
        container_name: daniel-test_api_pozos
        volumes:
            - '$PWD/student_age.json:/data/student_age.json'
        ports:
            - '4000:5000'
        image: 'daniel-pozos:V1'
```

- Cas 2:
```
docker run -d --network daniel-pozos_network --name Website-daniel_pozos -v $PWD/website:/var/www/html -p 80:80 php:apache
```
**docker-compose.yml** du **cas 2:
```
version: '3.3'
services:
    php:
        network_mode: daniel-pozos_network
        container_name: Website-daniel_pozos
        volumes:
            - '$PWD/website:/var/www/html'
        ports:
            - '80:80'
        image: 'php:apache'
```
### docker-compose.yml final

```
version: '3.3'
services:
    web-pozos1:
        image: 'php:apache'
        container_name: Website-daniel_pozos
        depends_on:
            - api
        volumes:
            - ./website:/var/www/html
        ports:
            - "8082:80"
        networks:
            - api_network_pozos
        environment:
            - USERNAME=toto
            - PASSWORD=python
        networks:
            - api_network_pozos
    api-pozos1:
        image: daniel-pozos:V1
        container_name: daniel-test_api_pozos
        ports:
            - "4000:5000"
        volumes:
            - ./simple_api/student_age.json:/data/student_age.json
        networks:
            - api_network_pozos


networks:
  api_network_pozos:
```

Une fois mon **docker-compose.yml** mise en place, je vais supprimer le docker de test **test_api_pozos** ainsi que le réseau créé **pozos_network**
```
docker ps -a
```
```
docker rm -f [id_docker_test]
```
```
docker network rm daniel-pozos_network
```
```
[vagrant@docker student-list]$ docker ps -a
CONTAINER ID   IMAGE             COMMAND                  CREATED       STATUS       PORTS                                       NAMES
b71d1d95885f   daniel-pozos:V1   "python ./student_ag…"   2 hours ago   Up 2 hours   0.0.0.0:4000->5000/tcp, :::4000->5000/tcp   daniel-test_api_pozos
```
```
[vagrant@docker student-list]$ docker rm -f b71d1d95885f
b71d1d95885f
```
```
[vagrant@docker student-list]$ docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```
```
[vagrant@docker student-list]$ docker network ls
NETWORK ID     NAME                   DRIVER    SCOPE
a51e9f3ce517   bridge                 bridge    local
faf514961911   daniel-pozos_network   bridge    local
32c790d7560b   host                   host      local
02b11bc7611e   none                   null      local
```
```
[vagrant@docker student-list]$ docker network rm daniel-pozos_network
daniel-pozos_network
```

j'ouvre le fichier **docker-compose.yml** avec l'éditeur texte *vi* puis je colle mon code.

```
vi docker-compose.yml
```

**Je vais lancer le déploiement de mon docker-compose**

Avant d'exécuter mon *docker-compose.yml*, je vais d'abord installer la commande **docker-compose**. Pour les instructions d'installation, cliquer le [lien](https://docs.docker.com/compose/install/).
```
$  sudo curl -L "https://github.com/docker/compose/releases/download/v2.4.1/docker-compose-linux-x86_64" -o /usr/local/bin/docker-compose
```
Je donne les droits à mon fichier.
```
sudo chmod +x /usr/local/bin/docker-compose
```
Je vérifie que **docker-composer** est bien installé.
```
docker-compose --version
```
```
Docker Compose version v2.4.1
```

**Déploiement proprement dite**
```
docker-compose up -d
```
Image téléchargée, réseau créé et mes deux containers créés. Rendu ci-dessous

![Image composeUp](/images/cmopose.png "Image du Run docker-compose")

Je vérifie que mes deux containers existent et qu'ils sont en mode **running**.

```
[vagrant@docker student-list]$ docker-compose ps
```
```
CONTAINER ID   IMAGE             COMMAND                  CREATED          STATUS          PORTS                                       NAMES
bbfadfcbc4c9   php:apache        "docker-php-entrypoi…"   37 seconds ago   Up 36 seconds   0.0.0.0:8082->80/tcp, :::8082->80/tcp       Website-daniel_pozos 
821c8666ff43   daniel-pozos:V1   "python ./student_ag…"   37 seconds ago   Up 36 seconds   0.0.0.0:4000->5000/tcp, :::4000->5000/tcp   daniel-test_api_pozos
```

Je vérifie que le réseau est bien créé
```
[vagrant@docker student-list]$ docker network ls
```
```
NETWORK ID     NAME                             DRIVER    SCOPE
98b7ca2ba69f   bridge                           bridge    local
dedef9dce2f6   host                             host      local
dcfc2cb9e6e6   none                             null      local
bc83b306b3c2   student-list_api_network_pozos   bridge    local
```

# images de test phase IAC à mettre

Pour faire les tests, je récupère l'ip de ma machine.

```
ip a
```
Puis je vais taper sur mon navigateur
```
http://192.168.56.37:8082
```
![Image Test](/images/imgWeb.png "Image Test Web")

Lorsque je clique sur le bouton **List Student**, j'ai le retour ci-dessous en image.

![Image Test](/images/imgWeb2.png "Image Test Web List Student")

**Fin de la phase de IAC, succès**

# Partie 3: Docker Registry

L'objectif de cette partie est de déployer un régistre privé afin d'enterriner notre image dans ce registre.

Pour ce faire, nous avons les indications dans l'énoncé du dépôt git du projet. Cliquer sur ce lien [registry](https://docs.docker.com/registry/) pour avoir la procédure à suivre.
Le lien suivant [interface](https://hub.docker.com/r/joxit/docker-registry-ui/) nous mène vers une image qui nous servira d'IHM pour notre **Registry**.

Commençons par déployer le **Registry**
- Mon **Registry** doit être déployé dans le même réseau **student-list_api_network_pozos**
- Je demarrer mon registry dans un container que je vais nommer **registry-pozos** avec le **tag** 2.
```
[vagrant@docker student-list]$ docker run -d -p 5000:5000 --name registry-pozos --network student-list_api_network_pozos registry:2
```
- Je vérifie que mon container **registry-pozos** est bien en place
```
[vagrant@docker student-list]$ docker ps -a
```

Résultat:

```
CONTAINER ID   IMAGE             COMMAND                  CREATED              STATUS              PORTS                                       NAMES
1a569eba1150   registry:2        "/entrypoint.sh /etc…"   About a minute ago   Up About a minute   0.0.0.0:5000->5000/tcp, :::5000->5000/tcp   registry-pozos
bbfadfcbc4c9   php:apache        "docker-php-entrypoi…"   31 minutes ago       Up 31 minutes       0.0.0.0:8082->80/tcp, :::8082->80/tcp       Website-daniel_pozos
821c8666ff43   daniel-pozos:V1   "python ./student_ag…"   31 minutes ago       Up 31 minutes       0.0.0.0:4000->5000/tcp, :::4000->5000/tcp   daniel-test_api_pozos
```

- Je vais renommer mon image pour qu'elle puisse aller taper le registry. Je liste d'abord les images existantes.

```
[vagrant@docker student-list]$ docker images
```
Résultat:
```
REPOSITORY     TAG           IMAGE ID       CREATED          SIZE
daniel-pozos   V1            db06773c9af8   56 minutes ago   1.13GB
php            apache        0a2504efce56   3 days ago       458MB
registry       2             2e200967d166   2 weeks ago      24.2MB
python         2.7-stretch   e71fc5c0fcb1   2 years ago      928MB
```

Pour renommer mon image, je procède comme suite:
```
[vagrant@docker student-list]$ docker image tag daniel-pozos:V1 localhost:5000/daniel-pozos:V1
```
Puis:
```
[vagrant@docker student-list]$ docker images
```
Résultat:

```
REPOSITORY                    TAG           IMAGE ID       CREATED             SIZE
daniel-pozos                  V1            db06773c9af8   About an hour ago   1.13GB
localhost:5000/daniel-pozos   V1            db06773c9af8   About an hour ago   1.13GB
php                           apache        0a2504efce56   3 days ago          458MB
registry                      2             2e200967d166   2 weeks ago         24.2MB
python                        2.7-stretch   e71fc5c0fcb1   2 years ago         928MB
```
Ci-dessous, le nouveau tag qui a le même id que l'image et le même size.

```
localhost:5000/daniel-pozos   V1            db06773c9af8   About an hour ago   1.13GB
```
C'est donc ce nouveau tag que je vais envoyer dans le **Registry** ci-dessous.

```
registry                      2             2e200967d166   2 weeks ago         24.2MB
```
Pour envoyer ce tag dans le container **Registry**, je vais le pusher.

```
[vagrant@docker student-list]$ docker push localhost:5000/daniel-pozos:V1
```
Push effectué avec succès
```
The push refers to repository [localhost:5000/daniel-pozos]
3cbf4b72ee29: Pushed
2d656e0a73c8: Pushed
73ba8562fa4e: Pushed
811b6c5694d4: Pushed
1855932b077c: Pushed
fa28e7fcadc2: Pushed
4427a3d9a321: Pushed
4a03ae8d3bee: Pushed
a9286fedbd63: Pushed
d50e7be1e737: Pushed
6b114a2dd6de: Pushed
bb9315db9240: Pushed
V1: digest: sha256:df8a716216da598bb1d10826d00514833a1d22c2738635373f6c603ee0edd945 size: 2854
```
Pour lancer l'image **Registry**.

```
[vagrant@docker student-list]$ docker run -d --name registry-pozos_UI --network student-list_api_network_pozos -p 4002:80 -e REGISTRY_TITLE="POZOS REGISTRY" -e REGISTRY_URL=http://registry-pozos:5000 -e DELETE_IMAGES=true joxit/docker-registry-ui:static
```
Je vérifie que mon container est présent.

```
[vagrant@docker student-list]$ docker ps -a
```
Résultat:
```
1511a4422a5b   joxit/docker-registry-ui:static   "/docker-entrypoint.…"   About a minute ago   Up About a minute   0.0.0.0:4002->80/tcp, :::4002->80/tcp       registry-pozos_UI
1a569eba1150   registry:2                        "/entrypoint.sh /etc…"   20 minutes ago       Up 20 minutes       0.0.0.0:5000->5000/tcp, :::5000->5000/tcp   registry-pozos
bbfadfcbc4c9   php:apache                        "docker-php-entrypoi…"   50 minutes ago       Up 50 minutes       0.0.0.0:8082->80/tcp, :::8082->80/tcp       Website-daniel_pozos
821c8666ff43   daniel-pozos:V1                   "python ./student_ag…"   50 minutes ago       Up 50 minutes       0.0.0.0:4000->5000/tcp, :::4000->5000/tcp   daniel-test_api_pozos
```

Mes containers sont bien en **up**.

Je vais aller sur le navigateur voir si j'ai bien mon **Registry private**.
```
http://192.168.56.37:4002
```
Résultat de mon image **daniel-pozos** pushée dans le réistre privé.

![Image Registry](/images/registry.png "Image Registry Réussi")

Pour voir le détail des information de cette image, je clique dessus.

![Image Registry Detail](/images/registryDetail.png "Image Registry Detail")

Je vais envoyer une autre image dans mon **Private Regstry**. 

Pour ce afire, je vais d'abord **tager** l'image en question avec cette commande.
- Je liste d'abord les images existantes afin de choisir une.

```
[vagrant@docker student-list]$ docker images
```
Résultat:
```
REPOSITORY                    TAG           IMAGE ID       CREATED             SIZE
daniel-pozos                  V1            db06773c9af8   About an hour ago   1.13GB
localhost:5000/daniel-pozos   V1            db06773c9af8   About an hour ago   1.13GB
php                           apache        0a2504efce56   3 days ago          458MB
registry                      2             2e200967d166   2 weeks ago         24.2MB
joxit/docker-registry-ui      static        c97caf4d3877   11 months ago       24.5MB
python                        2.7-stretch   e71fc5c0fcb1   2 years ago         928MB
```
- Je choisis celle ci-dessous.

```
joxit/docker-registry-ui      static        c97caf4d3877   11 months ago       24.5MB
```
- Puis je la **tag** en **static**.

```
[vagrant@docker student-list]$ docker image tag joxit/docker-registry-ui:static localhost:5000/joxit/docker-registry-ui:static
```

- Je liste les images pour voir le nouveau **tag**

```
[vagrant@docker student-list]$ docker images
```

Résultat:

```
localhost:5000/daniel-pozos               V1            db06773c9af8   2 hours ago     1.13GB
daniel-pozos                              V1            db06773c9af8   2 hours ago     1.13GB
php                                       apache        0a2504efce56   3 days ago      458MB
registry                                  2             2e200967d166   2 weeks ago     24.2MB
joxit/docker-registry-ui                  static        c97caf4d3877   11 months ago   24.5MB
localhost:5000/joxit/docker-registry-ui   static        c97caf4d3877   11 months ago   24.5MB
python                                    2.7-stretch   e71fc5c0fcb1   2 years ago     928MB
```

- Puis je push mon image taguée

```
[vagrant@docker student-list]$ docker push localhost:5000/joxit/docker-registry-ui:static
```
Résultat du push.

```
The push refers to repository [localhost:5000/joxit/docker-registry-ui]
edbe59d03810: Pushed
0d6fce342e85: Pushed
04aa2dc1cf79: Pushed
373c05b5af87: Pushed
e6a792f5b7f7: Pushed
4689e8eca613: Pushed
3480549413ea: Pushed
3c369314e003: Pushed
4531e200ac8d: Pushed
ed3fe3f2b59f: Pushed
b2d5eeeaba3a: Pushed
static: digest: sha256:2a0a24016554b15c242b51b0a7bbeee16ebe147776a667a15aa4404c712d70f5 size: 2609
```
![Image Registry 2](/images/registry2.png "Image Registry 2")

# Recommandations

Je recommande de le site de [cours hadrien pelissie](https://cours.hadrienpelissier.fr/02-docker/) afin de reforcer vos connaissance théoriques et pratiques en **Ansible**, **Docker** et **Kubernetes**.

Je vous recommande aussi les cours de [eazytraining](https://eazytraining.fr/parcours-devops/) en général et particulièrement le **Parcours DevOps**.

Pour le fonctionnement et la manipulation du réseau dans **Docker*, je vous conseille, pour les débutants de clique [ici](https://devopssec.fr/article/fonctionnement-manipulation-reseau-docker) pour lire.

# Liens

- Image php:apache sur [Docker Hub](https://hub.docker.com/_/php?tab=description).
- Installer [docker-compose](https://docs.docker.com/compose/install/).
- Repository git de l'application est fourni par [Dirane](https://github.com/diranetafen/student-list).
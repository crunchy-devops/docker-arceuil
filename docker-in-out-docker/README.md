# Docker-in-Docker (DinD)


## mise en oeuvre
Il est tout à fait possible de faire du Docker-in-Docker (DinD). 
C'est d'ailleurs une pratique courante, bien que technique, 
qui consiste à faire fonctionner un démon Docker (dockerd) à 
l'intérieur d'un conteneur lui-même géré par un Docker hôte.

Voici comment cela fonctionne, ses usages et ses limites.

1. Comment le mettre en œuvre ?
   Docker propose une image officielle dédiée à cet usage : ```docker:dind.```

Pour le lancer, il y a une condition indispensable : 
le conteneur doit être exécuté en mode privilégié (--privileged). 
Cela est nécessaire car le Docker "enfant" doit pouvoir monter des systèmes 
de fichiers (UnionFS) et manipuler les couches réseau, ce qui nécessite des
droits d'administration sur le noyau Linux.


## Docker out of Docker
### install gitlab-runner
```shell
docker run -d --name gitlab-runner --restart always \
  -v /srv/gitlab-runner/config:/etc/gitlab-runner \
  -v /var/run/docker.sock:/var/run/docker.sock \
  gitlab/gitlab-runner:latest
```
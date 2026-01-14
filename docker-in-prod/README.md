# install docker in prod
```shell
sudo dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo dnf update
sudo dnf install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
cd /home/alma/docker-arcueil
sudo fdisk -l
sudo mkfs.xfs /dev/sdc
sudo mkdir -p /mnt/data/docker
```

## bash completion
```shell
sudo dnf install -y docker-bash-completion
```
### change .bashrc
```shell
# add
cat <<EOT >> ~/.bashrc
if [ -f /etc/bash_completion.d/authselect-completion.sh ]; then
    . /etc/bash_completion.d/authselect-completion.sh
fi
EOT
```


## daemon.json   /etc/docker/daemon.json
```json
{
  "data-root": "/mnt/data/docker",
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m",
    "max-file": "3"
  },
  "live-restore": true,
  "userland-proxy": false,
  "no-new-privileges": true,
  "default-ulimits": {
    "nofile": {
      "Name": "nofile",
      "Hard": 64000,
      "Soft": 64000
    }
  }
}
```

## SElinux
```shell
# Installation de l'utilitaire de gestion de contexte
sudo dnf install -y policycoreutils-python-utils

# Appliquer le contexte SELinux au nouveau dossier
sudo mkdir -p /mnt/data/docker
sudo semanage fcontext -a -t container_var_lib_t "/mnt/data/docker(/.*)?"
sudo restorecon -Rv /mnt/data/docker
```

# seLinux-enabled: true
Pour éviter que vos conteneurs ne s'arrêtent de fonctionner brutalement après avoir activé la sécurité.

Lorsqu'on active `"selinux-enabled": true` dans le fichier `daemon.json`, Docker demande au noyau Linux (via SELinux) d'isoler strictement les processus des conteneurs du reste du système hôte et des autres conteneurs.

Voici ce que signifie **"si vos images (et vos configurations) sont prêtes"** :

### 1. La gestion des volumes (Le point le plus bloquant)

C'est le problème n°1. Si SELinux est activé, un conteneur n'a pas le droit de lire ou d'écrire dans un dossier monté depuis l'hôte (ex: `-v /data/config:/app/config`) sauf si ce dossier possède le bon **contexte de sécurité**.

* **Si ce n'est pas prêt :** Votre application affichera une erreur `Permission Denied`, même si vous êtes en `root` dans le conteneur.
* **La solution :** Vous devez ajouter le suffixe `:z` (partagé) ou `:Z` (exclusif) à vos montages de volumes pour que Docker change automatiquement le label SELinux :
* `docker run -v /data/config:/app/config:z my-image`



### 2. L'accès aux ressources système

Certaines images (outils de monitoring, agents de sauvegarde, cAdvisor) ont besoin d'accéder à `/var/run/docker.sock`, `/sys` ou `/proc`.

* **Si ce n'est pas prêt :** SELinux bloquera ces accès car il considère cela comme une tentative d'intrusion.
* **La solution :** Il faut soit utiliser des politiques SELinux spécifiques (modules `.pp`), soit lancer le conteneur avec un label de type "non-confiné" (`--security-opt label=disable`), ce qui réduit l'intérêt d'avoir activé SELinux globalement.

### 3. Les ports privilégiés

SELinux peut restreindre la capacité d'un processus conteneurisé à se lier à certains ports réseaux spécifiques s'ils ne sont pas étiquetés correctement.

### 4. L'utilisation d'utilisateurs non-root (Best Practice)

Si votre image Docker utilise un utilisateur spécifique (ex: `USER node`), SELinux sera encore plus pointilleux sur les droits d'accès aux fichiers. C'est une excellente pratique de sécurité, mais elle demande de tester que l'utilisateur a bien les droits de lecture sur les fichiers étiquetés pour Docker.

---

### En Résumé 

l'activation de SELinux dans Docker est une **protection "Targeted"** :

1. **Par défaut (false) :** Docker se repose sur les *Namespaces* et les *Cgroups* (isolation logique).
2. **Activé (true) :** Docker ajoute une couche de contrôle d'accès obligatoire (MAC).

**Conseil de déploiement :** En production sur AlmaLinux, on laisse généralement SELinux en mode `Enforcing` sur l'hôte, mais on ne force l'option `"selinux-enabled": true` dans Docker que si l'on maîtrise parfaitement le **labeling des volumes** (les fameux `:z` et `:Z`).

Si on l'active sans préparer les scripts de déploiement (Docker Compose), **tous les services avec des volumes persistants tomberont en erreur de permission.**


# lvm resize

```shell
sudo vgdisplay
sudo lvdisplay
sudo lvextend -l +100%FREE /dev/ubuntu-vg/ubuntu-lv
df -h
sudo resize2fs /dev/mapper/ubuntu--vg-ubuntu--lv
df -h # check 
```

# add external disk
```shell
sudo fdisk -l
sudo pvcreate /dev/sdb
sudo vgextend  ubuntu-vg  /dev/sdb
sudo lvextend -l +100%FREE /dev/ubuntu-vg/ubuntu-lv
sudo resize2fs /dev/mapper/ubuntu--vg-ubuntu--lv
df -h
```
# almalinux 10
```shell
sudo vgs
lvdisplay | grep "LV Path"
sudo lvdisplay | grep "LV Path"
sudo pvcreate /dev/sdb
sudo vgextend almalinux_device-140 /dev/sdb
sudo lvextend -l +100%FREE  /dev/almalinux_device-140/root
sudo xfs_growfs /
df -h
```

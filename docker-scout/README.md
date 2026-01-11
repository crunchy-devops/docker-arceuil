# Docker Scout
Docker Scout est l'outil indispensable à enseigner. Il marque le passage de "je sais faire un conteneur" à "je sais sécuriser une chaîne de production logicielle" (Sply Chain Security).

Voici un TP structuré pour maîtriser l'analyse de vulnérabilités, la conformité et la remédiation.

🎯 Objectifs du TP Docker Scout
Analyser une image existante pour identifier les CVE (Common Vulnerabilities and Exposures).

Comparer deux versions d'une image pour mesurer l'évolution du risque.

Remédier aux failles en changeant d'image de base ou en optimisant le Dockerfile.

Intégrer Scout dans un pipeline CI/CD (GitHub Actions ou Jenkins).


## Installe docker scout 
```shell
curl -sSfL https://raw.githubusercontent.com/docker/scout-cli/main/install.sh | sudo sh -s -- -b /usr/local/bin
sudo snf install wget
sudo dnf install wget

wget https://github.com/docker/sbom-cli-plugin/releases/download/v0.6.1/sbom-cli-plugin_0.6.1_linux_amd64.tar.gz
tar -xvxf sbom-cli-plugin_0.6.1_linux_amd64.tar.gz
sudo mv docker-sbom /usr/libexec/docker/cli-plugins/
sudo docker info
```
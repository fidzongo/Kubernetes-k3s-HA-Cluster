# k3s HA Cluster
Mise en place d'un clsuter kubernetes avec k3sup, k3s et le controleur ingress / loadbalancing par defaut (traefik)
<li>k3s: une version aléger de k8s et moins complexe a installer que k8s. Elle charge Kubernetes en simple binaire à lancer sur vos machines</li>
<li>k3sup: un utilitaire permettant d'automatiser l'installation de k3s. Il permet d'initier un cluster k3s, de joindre un cluster existant ou d'installer k3s sur une machine distante ou locale. Pour plus d'informations sur <a href="https://github.com/alexellis/k3sup">k3s/k3sup</a> </li>


# Contenu
<ul id="menu">
  <li><a href="https://github.com/fidzongo/Kubernetes-cluster-avec-k3sup-k3s-Traefik/tree/main#installation-de-k3sup" title="Installation de k3sup">Installation de k3sup</a></li>
  <li><a href="https://github.com/fidzongo/Kubernetes-cluster-avec-k3sup-k3s-Traefik/tree/main#mise-en-place-du-cluster-HA-k3s-avec-etcd" title="Mise en place du cluster HA k3s avec etcd">Mise en place du cluster k3s avec etcd</a></li>
  <li><a href="https://github.com/fidzongo/Kubernetes-cluster-avec-k3sup-k3s-Traefik/tree/main#mise-en-place-du-cluster-HA-k3s-avec-un-datastore-externe" title="Mise en place du cluster HA k3s avec un datastore externe">Mise en place du cluster k3s avec un datastore externe (mariadb)</a></li>
  <li><a href="https://github.com/fidzongo/Kubernetes-cluster-avec-k3sup-k3s-Traefik/tree/main#installation-de-k3s-dashboard" title="Installation de k3s dashboard">Installation de k3 dashboard</a></li>
  <li><a href="https://github.com/fidzongo/Kubernetes-cluster-avec-k3sup-k3s-Traefik/tree/main#installation-de-traefik-dashboard">Installation de traefik dashboard</a></li>
</ul>

# Installation de k3sup
```ruby
curl -sLS https://get.k3sup.dev | sh
sudo install k3sup /usr/local/bin/ # cette commande est nécessaire uniquement si l'installation est faite avec un utilisateur qui n'a pas de privilèges de copier le binaire k3s dans /usr/local/bin

# Pour verifier l'installation du binaire k3sup
k3sup --help
```

# Mise en place du cluster HA k3s avec etcd
ETCD est le datastore par defaut et intégré à k3s.

Pour mettre en place une haute disponibilité HA k3s il faut au minimum 3 noeuds masters. Pour plus d'informations voir <a href="[https://github.com/alexellis/k3sup](https://docs.k3s.io/)">la documentation officielle de k3s</a> </li>

Ce exemple d'ecrit un cluster à 6 noeuds :
- 3 serveurs (masters nodes - ubuntu)
- 3 agents (workers nodes - ubuntu)

```ruby
# Paramètres d'installation
server1=<hostname server 1>
server2=<hostname server 3>
server3=<hostname server 3>
agent1=<hostname agent 1>
agent2=<hostname agent 2>
agent3=<hostname agent 3>
username=<user ssh>

# Commandes a exécuter depuis une machine ou terminal ayant accès en ssh (avec échange de clés)aux serveurs du cluster
# Initialisation du premier master (master 1) du cluster
k3sup install --host ${server1} --user ${username} --cluster
# Installation master 2 (le serveur 2 rejoint le cluster initialement crée avec le serveur 1)
k3sup join --server-host ${server1} --host ${server2} --user ${username} --server
# Installation du master 3 (le serveur 3 rejoint le cluster initialement crée avec le serveur 1)
k3sup join --server-host ${server1} --host ${server3} --user ${username} --server

# Ajout du premier worker (worker1) au cluster
k3sup join --server-host ${server1} --host ${agent1} --user ${username}
# Ajout du worker 2 au cluster
k3sup join --server-host ${server1} --host ${agent2} --user ${username}
# Ajout du worker 3 au cluster
k3sup join --server-host ${server1} --host ${agent3} --user ${username}

# Pour verifier l'installation 
Depuis le terminal ou vous avez fait les installations un fichier de configuration (kubeconfig) a été automaiquement crée avec les informations du cluster (cluster, certificat, user, contexte...)
Il faut donc exporter ces paramètres pour pouvoir intérragir avec le cluster distant

export KUBECONFIG=/path_to/kubeconfig
kubectl config use-context default
kubectl get node -o wide
```
Pour plus d'informations sur l'installation vous pouvez consulter la page <a href="https://github.com/alexellis/k3sup">l'article d'alexellis</a>

# Mise en place du cluster HA k3s avec un datatastore externe
Nous utiliserons dans cette partie un datastore externe pour notre haute disponibilité k3s. l'avantage avec le datastore externe est que nous ppuvons faire une haute disponibilité avec au minimum deux noeuds masters.

Dans cet exemple un cluster à 3 noeurs (2 masters nodes et un worker node), nous utiliserons une base de données mysql déjà installée et configurée et que la chaine de connexion est connue.

```ruby
# export de la connexion à la base de donnée
export DATASTORE="mysql://<database-server>/defaultdb

# Génération de tocken (cette étape n'est pas nécessaire si vous avez déjà une installation d'un master existant. Alors le token peut être ecupéré dépuis la configuration du master /var/lib/rancher/k3s/server/token)
export TOKEN=$(openssl rand -base64 64)
export TOKEN=$(tr -dc A-Za-z0-9 </dev/urandom | head -c 64)

# Failing that, then try:
export TOKEN=$(head -c 64 /dev/urandom|shasum| cut -d - -f 1)

# Liste des serveurs / maters nodes
server1=vmubuntu1
server2=vmubuntu2

# Liste des serveurs / workers nodes
agent1=vmubuntu4

# utilisateur de connexion ssh
username=fidzongo

# Initialisation du premier master (master 1) du cluster
k3sup install --host ${server1} --user ${username} --datastore="${DATASTORE}" --token=${TOKEN}
# Installation master 2 (le serveur 2 rejoint le cluster initialement crée avec le serveur 1)
k3sup install --host ${server2} --user ${username} --datastore="${DATASTORE}" --token=${TOKEN}

# Ajout du premier worker (worker1) au cluster
k3sup join --server-host ${server1} --host ${agent1} --user ${username}
```

# Installation de k3s dashboard
Toujours depuis le serveur ou le terminal d'installation du cluster exécuter la commande ci-dessous pour installer le dashboard k3s:
```ruby
GITHUB_URL=https://github.com/kubernetes/dashboard/releases
VERSION_KUBE_DASHBOARD=$(curl -w '%{url_effective}' -I -L -s -S ${GITHUB_URL}/latest -o /dev/null | sed -e 's|.*/||')
sudo k3s kubectl create -f https://raw.githubusercontent.com/kubernetes/dashboard/${VERSION_KUBE_DASHBOARD}/aio/deploy/recommended.yaml

# Ajouter les ressources ci-dessous (users et privilèges d'accès au dashboard)
kubectl create -f dashboard.admin-user.yml -f dashboard.admin-user-role.yml

# Pour se connecter au dashboard exécuter la commande ci-dessous:
kubectl proxy

# Token de connexion au dashboard
kubectl -n kubernetes-dashboard create token admin-user

# URL de connexion au dashboard
http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
```

# Installation de traefik dashboard
Installation du dashboard traeffik
```
kubectl apply -f traefik-dashboard.yml
```

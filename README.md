# Kubernetes cluster avec k3sup / k3s / Traefik
Mise en place d'un clsuter kubernetes avec k3sup, k3s et le controleur ingress / loadbalancing par defaut (traefik)
<li>k3s: une version aléger de k8s et moins complexe a installer que k8S. Elle charge Kubernetes en simple binaire à lancer sur vos machines</li>
<li>k3sup: un utilitaire permettant d' automatiser l'installation de k3s. Il permet d'initier un cluster k3s, de joindre un cluster existant ou d'installer k3s sur machine distante ou locale. Pour plus d'informations sur <a href="https://github.com/alexellis/k3sup">k3s/k3sup</a> </li>


# Contenu
<ul id="menu">
  <li><a href="https://github.com/fidzongo/Kubernetes-cluster-avec-k3sup-k3s-Traefik/tree/main#installation-de-k3sup" title="Installation de k3sup">Installation de k3sup</a></li>
  <li><a href="https://github.com/fidzongo/Kubernetes-cluster-avec-k3sup-k3s-Traefik/tree/main#mise-en-place-du-cluster-k3s" title="Mise en place du cluster k3s">Mise en place du cluster k3s</a></li>
  <li><a href="https://github.com/fidzongo/Kubernetes-cluster-avec-k3sup-k3s-Traefik/tree/main#installation-de-k3s-dashboard" title="Installation de k3s dashboard">Installation de k3 dashboard</a></li>
  <li><a href="https://github.com/fidzongo/Kubernetes-cluster-avec-k3sup-k3s-Traefik/tree/main#installation-de-traefik-dashboard">Installation de traefik dashboard</a></li>
</ul>

# Installation de k3sup
```ruby
curl -sLS https://get.k3sup.dev | sh
sudo install k3sup /usr/local/bin/ # cette commande est nécessaire uniquement si l'installation est faite avec un utilisateur qui n'a pas de privilèges de copier le binaire k3s dans /usr/local/bin
```

k3sup --help

# Mise en place du cluster k3s
Ce exemple d'ecrit un cluster à 5 noeuds :
- 2 serveurs (masters nodes)
- 3 agents (workers nodes)

```ruby
# Paramètres d'installation
server1=<your server 1 hostname>
server2=<your server 2 hostname>
agent1=<your agent 1 hostname>
agent2=<your agent 2 hostname>
username=<your ssh username>

# Commandes a exécuter depuis une machine ou terminal ayant accès en ssh (avec échange de clés)aux serveurs du cluster
# Pour initialiser le cluster (installation du serveur/master 1) 
k3sup install --host ${server1} --user ${username} --cluster
# Installation du serveur/master 2 (le serveur 2 rejoint le cluster initialement crée avec le serveur 1)
k3sup join --server-host ${server1} --host ${server2} --user ${username} --server
# Pour ajouter un agent/worker 1 au cluster
k3sup join --server-host ${server1} --host ${agent1} --user ${username}
# Pour ajouter un agent/worker 1 au cluster
k3sup join --server-host ${server1} --host ${agent2} --user ${username}

# Pour verifier l'installation (depuis le terminal ou vous avez fait les installations)
export KUBECONFIG=/path_to/kubeconfig
kubectl config use-context default
kubectl get node -o wide

```
Pour plus d'informations sur l'installation vous pouvez consulter <a href="https://github.com/alexellis/k3sup">l'article d'alexellis</a>

# Installation de k3s dashboard
Toujours depuis le serveur ou le terminal d'installation du cluster exécuter la commande ci-dessous pour installer le dashboard k3s:
```ruby
GITHUB_URL=https://github.com/kubernetes/dashboard/releases
VERSION_KUBE_DASHBOARD=$(curl -w '%{url_effective}' -I -L -s -S ${GITHUB_URL}/latest -o /dev/null | sed -e 's|.*/||')
sudo k3s kubectl create -f https://raw.githubusercontent.com/kubernetes/dashboard/${VERSION_KUBE_DASHBOARD}/aio/deploy/recommended.yaml

kubectl create -f dashboard.admin-user.yml -f dashboard.admin-user-role.yml

kubectl -n kubernetes-dashboard create token admin-user

kubectl proxy
```

# Installation de traefik dashboard

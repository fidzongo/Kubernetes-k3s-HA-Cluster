# Kubernetes-cluster-avec-k3sup-k3s-Traefik
Mise en place d'un clsuter kubernetes avec k3sup, k3s et le loadbalancing par defaut (traefik)
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
sudo install k3sup /usr/local/bin/
```

k3sup --help

# Mise en place du cluster k3s
Cet exemple d'ecrit un cluster à deux noeuds et deux workers nodes.
Pour plus d'informations sur l'installation vous pouvez consulter <a href="https://github.com/alexellis/k3sup">l'article d'alexellis</a>
## Master - node01
sudo k3sup install --host server1 --user fidzongo --cluster
## Master - node02
## Worker - worker01
## Worker - worker02

# Installation de k3s dashboard

# Installation de traefik dashboard

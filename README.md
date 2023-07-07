# Brief20
Mini Procédure pour AKS Monitoring et RandomLogger

Créer un cluster AKS Simple free avec une seule node en autoscaling 
(En Azure CNI, il faudra plus tard autoriser l’accès depuis Internet aux IP Externe des services Prometheus et Graphana que l’on active)
Monitoring peut se faire depuis Azure Monitor et/ou Log Analytic Workspace
Sur une invite de commande (PS 7) : 
Installer chocolatey
Installer Helm
Installer k9s
Installer kubens et kubectx
(Se connecter au cluster aks) (faire un az connect a la sub dans lequel le cluster AKS se situe)
az aks get-credentials -n AKS-CRO-B20 -g PERSO_CLEMENT


Appli générant des logs exemple : 
https://hub.docker.com/r/chentex/random-logger/
Le déploiement des containers pour cette appli a été fais directement sur le cluster AKS sur le portail Azure : 

Créer un namespace monitoring
Prometheus = collecte les metrics uniquement (fonctionne pour container et VM mais ici on ne l’utilise que pour les container)
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install prometheus prometheus-community/prometheus
Graphana = Affiche les infos en prenant des data sources (ajouter des datasources pour Prometheus et Elasticsearch sur graphana plus tard) 
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
helm install grafana grafana/grafana
Loki et Promtail = collecte les logs 
helm install loki grafana/loki-stack 
tuto loki promtail : 
https://cylab.be/blog/197/deploy-loki-on-kubernetes-and-monitor-the-logs-of-your-pods

taper : k9s afin d’avoir une meilleure interface du cluster AKS.
Dans k9s, taper : <svc> ou <ns> ou <pods> etc…

Voir pour modifier les conf de base (dashboard)

ATTENTION LES DATASOURCES sont à refaire a chaque fois que le AKS redémarre !!!
Trouver un moyen pour enregistrer la conf grafana des dashboard afin qu’elle reste
https://grafana.com/docs/grafana/latest/administration/provisioning/#using-environment-variables
https://grafana.com/docs/grafana/latest/setup-grafana/configure-grafana/

Vidéo YT utile :
https://www.youtube.com/watch?v=gDX3HtoRta4&list=PLpbcUe4chE79sB7Jg7B4z3HytqUUEwcNE&index=15



Ensuite, il faut exposer les “services” Prometheus et Graphana (et Loki j’imagine) en les transformant de type “Cluster IP” au type “LoadBalancer” (soit avec une commande soit en modifiant le manifest yaml directement grâce a k9s)

Des External IP apparaissent ensuite.
Pour être sûr vérifier le nsg lié au Vnet : 
Ajouter des règles au NSG lié au Subnet afin d’autoriser l’accès depuis Internet aux IP  Externe de Prometheus et Graphana (et Loki ?)


kubens permet de savoir dans quel namespace on se trouve

Ensuite, on a normalement accès aux 2 IP externe en les tapant dans la barre de recherche, si la page ne charge ou qu’il y a un soucis, c’est un problème de réseau (NSG)

On ajoute ensuite les datasources nécessaire en se connectant à l’interface Graphana.

(pour avoir les identifiants admin graphana c’est :
admin et pour le mdp : 
kubectl get secret --namespace monitoring grafana -o jsonpath="{.data.admin-password}" | base64 --decode
si ca ne fonctionne pas, enlever la partie après le pipe et prenez le mdp en base64 puis décoder grâce à un site Internet ou autrement pour avoir le mot de passe admin de graphana

Ensuite il faut se créer ou importer des Dashboard officiel ou fait par la commu Graphana

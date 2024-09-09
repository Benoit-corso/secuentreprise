<div align="center">
  <img src="https://i.giphy.com/media/v1.Y2lkPTc5MGI3NjExZTkzMXIycGx2bWtuOWkyOWZvd3Z2dGd2dnBvaG9ieDhiZDcxa3IzdiZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/QpVUMRUJGokfqXyfa1/giphy.gif" alt="File Sharing GIF" width="75%" height="500" style="">
  <h1 style="">Secu Entreprise</h1>
  <p>Projet de gestion integrant Monitoring, Supervision, et Administration des infrastructures. La collecte des données permet une d'agir de facon proactive.</p>

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
![GitHub contributors](https://img.shields.io/github/contributors/benoit-corso/secuentreprise)
</div>
<blockquote>
Benoit CORSO<dd>benoit.corso@laplatforme.io</dd>
Mathias PUTZOLA<dd>mathias.putzola@laplateforme.io</dd>
Renova MANIRAFASHA<dd>renova.manirafasha@laplateforme.io</dd>
</blockquote>

## Prerequisite
```
debian 12 (bookworm) arm64 - Raspberry Pi 4
- zabbix (version: 7.0 LTS)
- php-fpm (version: 8.2)
- prometheus (version: 2.54.1)
- node_exporter (version: 1.8.2)
- windows_exporter (version: 0.28.1)
- process_exporter (version: 0.8.3)
- grafana (version: 11.2.0)
- elasticsearch (version: 7.17.23)
- logstash (version: 7.17.23)
- kibana (version: 7.17.23)
- metricbeat (version: 7.17.23)
- thehive (version: ???)
- MISP (version: ???)
- nginx (latest)
```

### Installation Informations
Nous avons mis en place une redirection par upstream des services auxquels nous voulons acceder par une redirection dns vers un domaine de notre choix.
Puis Nginx se charge de recuperer les upstreams des differents services en local et de servir le contenu sur des VHost normalisé ainsi l'acces auxx services est facilité et la securité renforcée car les differents services ne sont pas exposés sur le reseaux, ce qui evite des fuites de données qui pourrais servir a de la reconnaissance de notre infrastructure.

Voici donc les adresses de nos services disponibles:
```
zabbix.nox-corp.net
grafana.nox-corp.net
kibana.nox-corp.net
```

### Zabbix
La configuration des alertes a ete faite par mail. En changeant les macros de l'hote associés nous avons defini les trigger a 75% de charge pour le cpu et la memoire, sur une duree defini de 2min.
Pour ce faire il faut aller dans Collecte de donnees > Hote > declencheurs.
- Choisir l'evenement associé puis verifier le declencheur parents, passer la variable au temps voulu.
Puis changer les macros concernant le niveau de charge de travail:
- Hote > macro > ajouter:
    - {$CPU.UTIL.CRIT}      = 75
    - {$IF.UTIL.MAX}        = 75
    - {$MEMORY.UTIL.MAX}    = 75
Enfin, aller dans Utilisateurs > Utilisateurs > Admin > Media:
    Puis ajouter le media correspondant, dans le cas present pour les mails il suffit d'entrer les adresses mail des administrateurs.

### Prometheus
Premierement il faut configurer prometheus sur le port souhaité, en tant que service, dans le cas present nous somme sur une machine debian 12.
Il faut donc creer un 'unit file' pour systemd.
Une fois l'unit-file configuré nous devons ouvrir le fichier de configuration de prometheus afin qu'il puisse recuperer les données de node_exporter, tout comme d'autres exporters.
Dans notre situation, etant donné que toutes l'infra du projet est hebergée sur un raspberry pi. J'ai pris la decision d'ajouter windows_exporter et process_exporter afin de collecter plus de données.
Egalement, node_exporter est configuré afin d'activer des collecteurs supplementaires, le services est donc lancé avec les collecteurs par defaut plus ceux-ci qui sont passé en tant qu'arguments dans la ligne de commande du service:
    --collector.
        - processes.
        - systemd.
        - network_route.
        - cpu_vulnerabilites.
il faut egalement configurer prometheus pour qu'il aille taper dans les données des collecteurs.
/!\ Fournir le fichier de configuration /!\

### Grafana
Le fonctionnement et l'installation de grafana est assez simple tout se fait par l'interface, le travail a partir de ce moment la est axé sur la selection et la normalisation des données. Il faut egalement avoir des petites notions de certaines fonctions (et donc d'anglais ofc) afin de pouvoir normaliser et presenter les données de facon intuitif.
Comme nous avons fait le choix d'integrer des données qui sont utile dans la gestion de notre infrastructures il faut a present les integrés et les présenter.

### ELK (Elastic LogStash Kibana)
ElasticSearch
La première étape consiste à installer ElasticSearch, qui servira d'intermédiaire entre logstach et kibana. Sur notre machine Debian 12, ElasticSearch doit être configuré pour fonctionner sur le port 9200.
Installation et configuration :
        - Ajout de la clé publique et repository d'Elastic
	curl -fsSL https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
	sudo apt-get install apt-transport-https
	echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list
	- Installation du paquet : 
	sudo apt update
	sudo apt install elasticsearch
	- Configuration :
		- Modifier le fichier de configuration /etc/elasticsearch/elasticsearch.yml pour définir le cluster.name, le node.name et d'autres paramètres essentiels.
		- S'assurer que le service est bien démarré et écoute sur le bon port en exécutant systemctl enable --now elasticsearch.

Logstach
Logstash est utilisé pour ingérer, transformer et envoyer les logs vers ElasticSearch. Sa configuration se fait par un fichier .conf qui décrit comment les logs sont collectés et traités.
Installation et configuration :
	- Installer Logstash avec le package manager.
		sudo apt install logstash
	- Créer un fichier de configuration, par exemple /etc/logstash/conf.d/logstash.conf qui inclut les sections suivantes :
		sudo nano /etc/logstach/conf.d/logstach.conf
		- Input : Données collectées de fichiers locaux ou d'autres sources comme Filebeat (Port 5044).
		- Filter : Transformations des logs. (CSV / GROK)
		- Output : Envoie des données à ElasticSearch (Port 9200).

Filebeat
Filebeat est un agent léger de collecte de logs développé par Elastic, qui envoie les fichiers de logs de manière continue vers Logstash ou directement vers ElasticSearch. Son rôle est de simplifier la collecte de logs sur des machines distantes, ce qui en fait un outil essentiel pour l'observation de systèmes distribués.
Installation et configuration :
	- Installation du paquet :
		sudo apt update
		sudo apt install filebeat
	- Configuration du fichier filebeat.yml :
		sudo nano /etc/filebeat/filebeat.yml
		- Input Données collectées localement :
	 		filebeat.inputs:
  		  	- type: log
   			enabled: true
		- Accès au fichier :
   			paths:
     	  		- /path/to/*.log
		- Ouput : 
 			output.elasticsearch:
 			  hosts: ["http://localhost:9200"]

Metricbeat
Metricbeat est un agent léger utilisé pour collecter des métriques système et de services sur vos serveurs et applications. Il fait partie de la suite Beats d'Elastic, qui comprend différents agents spécialisés pour la collecte de données. Metricbeat surveille l'état des ressources (CPU, mémoire, disque, réseau, etc.) ainsi que les services tels que Docker, Kubernetes, MySQL, NGINX, etc. Ensuite, il envoie ces métriques vers des outils comme ElasticSearch ou Logstash pour un traitement et une visualisation dans Kibana.
Installation et configuration : 
	- Installation du paquet : 
		sudo apt-get install metricbeat
	- Configuration du fichier metricbeat.yml
		sudo nano /etc/metricbeat/metricbeat.yml
		- Utilisation de modules : 
 			metricbeat.config.modules:
 			  path: ${path.config}/modules.d/*.yml
 			metricbeat.modules:
  			 - module: system
   			   metricsets:
      				- cpu
 			        - memory
      				- network
      				- filesystem
      				- process
      				- diskio
   			   enabled: true
		- Output :
 			output.elasticsearch:
 			   hosts: ["http://localhost:9200"]
 			setup.kibana:
  			   host: "localhost:5601"


Kibana
Kibana sert d'interface pour visualiser et explorer les données stockées dans ElasticSearch.
Installation et configuration :
	- Installer Kibana avec la package manager :
		sudo apt install kibana
	- Configuration du fichier Kibana.yml
		sudo nano /etc/kibana/kibana.yml
		- Écoute sur l'interface réseau : 
			server.host: "0.0.0.0" 
		- Écoute du port ElasticSearch :
			elasticsearch.hosts: ["http://localhost:9200"]


Tout les services ont été démarré avec la commande : 
	sudo systemctl enable [name.service]
	sudo systemctl start [name.service]
Et pourront être vérifié avec la commande : 
	sudo systemctl status [name.service]
Avec un accès aux logs avec la commande :
	sudo tail -f /var/log/[name.service]


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


### ELK-Watcher


### The Hive & MISP


## Deroulement du projet
Etape 1: Ok.
Etape 2: Ok.
Etape 3: Ok.
Etape 4: Ok.
Etape 5: Ok.
Etape 6: Ok. (need extract forward to ELK ?? i think)
Etape 7: Ko.
Etape 8: Ko.
Etape 9: Ko.

#### Windows Exporter
install service with:
```
msiexec /i windows_exporter-0.28.1-amd64.msi ENABLED_COLLECTORS="os,system,cpu,net,ad,iis,logon,memory,process,tcp,textfile,thermalzone,service,physical_disk" LISTEN_PORT=9182
# Open firewall
New-NetFirewallRule -DisplayName "Allow Inbound Windows Exporter" -Direction Inbound -Program "C:\Program Files\windows_exporter\windows_exporter.exe" -RemoteAddress LocalSubnet -Action Allow
```

#### TODO:
- Check every binding address for services, should be all binded to 127.0.0.1
- Check every VHosts.
- Finished layout and connected services for displaying datas. (done on grafana, waiting for kibana)
- Maybe connect zabbix alerts:
                - w/ windows notifications.
                - w/ SMS.
                - into Grafana.
- Faire ELK-watcher, pour, je pense, recuperer les datas des packets capturés avec wireshark.
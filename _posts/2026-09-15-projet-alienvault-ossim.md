---
title: "Déploiement d'une plateforme SIEM AlienVault OSSIM et simulation d'attaques"
date: 2026-09-15 10:00:00 +0000
categories: [SOC, Blue Team]
tags: [alienvault, ossim, siem, ossec, nmap, brute-force, détection-intrusion]
description: "SIEM OSSIM avec agents OSSEC et simulation d'attaques pour valider la détection."
pin: true
image:
  path: /assets/img/ossim-cover.jpg
  alt: Logo AlienVault OSSIM
---

## Contexte

Le projet  réalisé dans le cadre de ma formation, avec pour objectif de construire un environnement SOC miniature de bout en bout : déployer un SIEM open source, y récolter des logs réels, puis générer volontairement des attaques pour vérifier que la plateforme les détecte correctement.

> **Périmètre pédagogique :** l'organisation et les données décrites dans ce projet sont fictives.
{: .prompt-info }
## Objectifs

- Déployer et configurer **AlienVault OSSIM** sur un serveur virtuel VMware
- Intégrer des agents **OSSEC** sur des machines Windows et Linux pour la remontée centralisée des logs
- Simuler des scénarios d'attaque réalistes pour tester la chaîne de détection
- Analyser les événements de sécurité générés et produire des rapports d'incidents exploitables

## Environnement

| Élément                  | Détail                                 |
| ------------------------ | -------------------------------------- |
| Hyperviseur              | VMware workstation pro                 |
| Plateforme SIEM          | Alien Vault OSSIM v5.8.16              |
| Agents de collecte       | OSSEC (HIDS) sur Windows et Linux      |
| Outils d'attaque simulée | KALI Linux (Hydra, Nmap, nikto, Hping) |
| adresse reseau           | 10.0.254.0/24                          |


![L'architecture simplifiée Alien Vault OSSIM](/assets/img/L'architecture simplifiée Alien Vault OSSIM v5.8.16.png)


## Mise en œuvre

### 1. Installation et déploiement d'OSSIM

Déploiement de la machine virtuelle AlienVault OSSIM sous VMware, configuration réseau initiale (interface de management dédiée), puis configuration via l'assistant web de première connexion (plage réseau à surveiller, fuseau horaire, etc.).

Voici les ressources allouées des machines de notre lab :
Serveur OSSIM :
- Système d’exploitation : Alien Vault OSSIM v5.8.16
- Mémoire RAM : 8gb
- Stockage : 40gb
Les machines clientes :
- Système d’exploitation : Windows 10 pro, Ubuntu Desktop 22.4
- Mémoire RAM : 2g
- Stockage :20gb


### 2. Intégration des agents OSSEC

Installation de l'agent **OSSEC** sur les machines à surveiller :
- Windows 10 pro : installation de l'agent, configuration du serveur OSSIM comme manager, activation de la remontée des logs d'événements Windows (Sécurité, Système).

![Configuration de l'agent ossec sur win 10](/assets/img/Config Agent OSSEC.png)

- Ubuntu Desktop 22.4 : installation de l'agent via le gestionnaire de paquets, configuration de `ossec.conf` pour pointer vers le serveur OSSIM, vérification de la remontée des logs `auth.log`/`syslog`.

![Configuration de l'agent ossec sur ubuntu](/assets/img/Config Agent OSSEC ubuntu.png)

Vérification que chaque agent apparaît bien comme **actif** dans le tableau de bord OSSIM (Environment > Detection > HIDS).



### 3. Scénarios d'attaque simulés

Pour vérifier que la chaîne de détection fonctionne réellement, deux scénarios ont été menés depuis une machine attaquante Kali dédiée :

**uniquement en environnement de lab isolé**

- **Brute-force SSH avec Hydra** : tentatives de connexion répétées sur le service SSH d'une machine Linux surveillée, pour déclencher la détection de force brute par OSSIM/OSSEC.
- **Scan réseau avec Nmap** : scan de ports sur le réseau simulé pour vérifier la détection de reconnaissance réseau.`nmap -sS -p- 10.0.254.20`
- **Scan de vulnérabilités web avec Nikto** : scan d'un serveur web du réseau simulé pour repérer des fichiers/configurations sensibles exposés, et vérifier que ce type de reconnaissance applicative est lui aussi détecté. `hydra -l user -P wordlist ssh://10.0.254.24`
- **Simulation de DDoS avec hping3** : envoi d'un volume élevé de paquets  vers une machine cible du réseau simulé, pour tester la détection d'un pic de trafic anormal.`hping3 -S --flood -p 80 10.0.254.20`



### 4. Scan de vulnérabilités du réseau

En complément des attaques simulées, un scan de vulnérabilités a été mené sur trois machines de test du réseau simulé, pour identifier les failles propres à ces   machines indépendamment d'un scénario d'attaque précis.


- Scan de chaque machine (Windows et Linux) pour recenser les vulnérabilités connues (CVE), les services obsolètes, et les mauvaises configurations.
- Priorisation des vulnérabilités trouvées par niveau de criticité.

![scan de vulnerabilite des machines du reseau](/assets/img/scan de vulnerabilite des machines du reseau.png)



### 5. Configuration des règles de détection et de corrélation par type d'attaque

Chaque scénario a nécessité de vérifier ou créer une règle de corrélation dédiée dans OSSIM, pour que le comportement remonte comme un événement de sécurité qualifié :

| Attaque simulée                    | Logique de détection                                                                                                        | Source de log utilisée                    |
| ---------------------------------- | --------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------- |
| Brute-force SSH (Hydra)            | Multiples échecs d'authentification sur un même compte/IP en peu de temps                                                   | Logs OSSEC (`auth.log` via l'agent Linux) |
| Scan réseau (Nmap)                 | Connexions à de nombreux ports différents depuis une même IP en peu de temps                                                | Logs réseau / détecteur de scan OSSIM     |
| Scan de vulnérabilités web (Nikto) | Volume élevé de requêtes HTTP suspectes (user-agent Nikto, chemins sensibles testés) en peu de temps depuis une même source | Logs du serveur web collectés par l'agent |
| DDoS (hping3)                      | Pic anormal de paquets/connexions vers une même cible sur une courte période                                                | Logs réseau / NetFlow ou équivalent       |

Ces règles existent déjà et peuvent être modifiées pour correspondre au degré de détection requis dans notre structure

![regle de corelation des logs](/assets/img/regle de corelation des logs.png)

## Résultats et analyse

- Le brute-force SSH a généré une alerte **niveau de risque elevé** dans OSSIM.
  ![Resultat Attaque SSH Brutforce](/assets/img/Resultat Attaque SSH Brutforce.png)
- Le scan Nmap a été détecté par la règle de **network scan detection**, avec identification de l'IP attaquante et des ports scannés.
  ![Resultat SCAN nmap](/assets/img/Resultat SCAN nmap.png)
- Le scan Nikto a **été détecté**.
 ![Resultat SCAN nikto](/assets/img/Resultat SCAN nikto.png)
- La simulation DDoS avec hping3 a **généré une alerte de type flood**.
  ![Resultat Attaque DDOS](/assets/img/Resultat Attaque DDOS.png)
- Analyse des événements dans le SIEM : corrélation entre les logs OSSEC bruts et les événements de sécurité générés par le moteur de corrélation OSSIM.



## Compétences démontrées

- Déploiement et configuration d'une plateforme SIEM open source (AlienVault OSSIM)
- Intégration d'agents HIDS multiplateforme (OSSEC Windows/Linux)
- Conduite de scénarios d'attaque contrôlés (brute-force, scan réseau, scan de vulnérabilités web, DDoS) pour valider une chaîne de détection
- Rédaction de règles de corrélation adaptées à chaque type d'attaque
- Réalisation et lecture de scans de vulnérabilités réseau
- Analyse d'événements de sécurité 

## Conclusion et pistes d'amélioration

Avec ses diverses fonctionnalités, OSSIM se présente comme une solution complète de détection des intrusions, de gestion des évènements de sécurité ainsi que de corrélation des logs. De plus, cette solution est évolutive avec la mise en place d'Open Threat Exchange (OTX) qui continue de mettre à jour la base de données des CVE d'Alien Vault OSSIM. Bien qu'efficace pour détecter les incidents, il reste limite en ce qui concerne le confinement puis l'éradication.


---

*Projet réalisé dans le cadre de ma formation, en environnement de test isolé (VMware), sans exposition à un réseau de production.*
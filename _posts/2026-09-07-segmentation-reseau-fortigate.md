---
title: "Segmentation réseau et contrôle des accès avec FortiGate"
date: 2026-09-07 00:00:00 +0000
categories: [network security, firewall]
tags: [fortigate, vlan, firewall, network-security, segmentation, troubleshooting]
pin: true
image:
  path: /assets/img/Forti.png
  alt: Logo Forti
---

# Segmentation réseau et contrôle des accès avec FortiGate

## Contexte

Dans le cadre de mes activités professionnelles en environnement de microfinance, je suis amené à intervenir sur des problématiques liées au réseau, à la sécurité des accès et au support des utilisateurs.
L'un des sujets sur lesquels j'ai travaillé concerne la **segmentation du réseau à l'aide de VLAN et le contrôle des communications avec un pare-feu FortiGate**.
L'objectif était de mieux séparer les différents types de flux réseau tout en conservant les communications nécessaires au fonctionnement des services.

> **Note de confidentialité :** les informations présentées ici sont volontairement génériques. Les adresses IP, noms de VLAN, noms de machines, règles de sécurité et autres informations internes ne sont pas publiés.
{: .prompt-info }

## Objectifs

- segmenter le réseau à l'aide de VLAN ;
- contrôler les communications inter-VLAN ;
- appliquer des politiques de sécurité au niveau du FortiGate ;
- contrôler l'accès à certains réseaux et services ;
- mettre en place un portail captif sur un réseau dédié ;
- analyser le trafic lorsqu'une communication ne fonctionnait pas comme prévu ;
- améliorer ma méthodologie de diagnostic réseau.

## Architecture générale

L'architecture peut être représentée de manière simplifiée comme suit :

![Segmentation réseau avec FortiGate](/assets/img/L'architecture simplifiée(1).png)



Le FortiGate joue un rôle central dans cette architecture :

- passerelle des différents réseaux ;
- filtrage des communications ;
- application des politiques de sécurité ;
- observation et analyse du trafic ;
- contrôle des accès selon le réseau source et la destination.

## Mise en place de la segmentation

La première étape consiste à définir les différents segments réseau selon les besoins de l'organisation.

Chaque VLAN possède son propre sous-réseau et sa propre interface logique sur le FortiGate.

Cette séparation permet notamment de limiter les communications inutiles entre les postes utilisateurs, les serveurs et les réseaux destinés à des usages spécifiques.

L'idée n'est pas simplement de créer plusieurs VLAN, mais de définir clairement :

**Qui peut communiquer avec qui, vers quelle destination et sur quels services ?**

## Politiques de sécurité sur le FortiGate

Une fois les interfaces et les réseaux définis, les communications inter-VLAN sont contrôlées par les politiques de sécurité du FortiGate.

Une politique peut être pensée selon le modèle :



## Politiques de sécurité sur le FortiGate

Les règles de filtrage sont définies selon le principe de segmentation des différents
réseaux et du moindre privilège. Chaque VLAN dispose uniquement des accès nécessaires
à son usage.

| ID  | Source | Destination | Action | Journalisation | Objectif |
| --- | ------ | ----------- | ------ | -------------- | -------- ||
| FW-01 | VLAN 1 - Utilisateurs | Internet              | ACCEPT       | Oui            | Permettre l'accès Internet aux postes utilisateurs                          |
| FW-02 | VLAN 1 - Utilisateurs | VLAN 2 - Serveurs     | ACCEPT       | Oui            | Permettre l'accès aux services internes                                     |
| FW-03 | VLAN 1 - Utilisateurs | VLAN 3 - AP           | Selon besoin | Oui            | Contrôler les communications avec le réseau AP                              |
| FW-04 | VLAN 2 - Serveurs     | Internet              | ACCEPT       | Oui            | Permettre aux serveurs d'accéder aux services Internet nécessaires          |
| FW-05 | VLAN 2 - Serveurs     | VLAN 1 - Utilisateurs | ACCEPT       | Oui            | Permettre aux utilisateurs d'accéder aux services hébergés sur les serveurs |
| FW-06 | VLAN 2 - Serveurs     | Internet              | DENY         | Oui            | Empêcher l'accès direct aux serveurs depuis Internet                        |
| FW-07 | VLAN 3 - AP           | Internet              | ACCEPT       | Oui            | Permettre l'accès Internet aux clients connectés aux AP                     |
| FW-08 | VLAN 3 - AP           | VLAN 2 - Serveurs     | DENY         | Oui            | Isoler les clients Wi-Fi des serveurs internes                              |
| FW-09 | VLAN 3 - AP           | VLAN 1 - Utilisateurs | DENY         | Oui            | Isoler le réseau AP du réseau utilisateurs                                  |
| FW-10 | Internet              | VLAN 2 - Serveurs     | DENY         | Oui            | Bloquer les connexions initiées depuis Internet vers les serveurs           |

>Toute les regles avec deny sont par defaut bloque par implicit deny mais pour plus de visibilite sur ce que nous vouslons nous les ajoutons
{: .prompt-info }

Cette approche permet d'appliquer le principe du **moindre privilège** : un poste ou un réseau ne doit pas avoir accès à davantage de ressources que nécessaire.

### Illustration
Pour le post il n'y a pas moyen d'utiliser un forti en prod. Nous allons donc utiliser un forti virtuel installe sur vmware et active avec une licence d'evaluation qui est vraiment limite, la creation des vlan et des des regles de pare est restreinte. Ce sera juste assez pour une illustration.
Etant limite par notre forti nous allons utiliser les 3 portrs mis a notre disposition comme suit pour creer les regles du pare feu : 
- le port 1 sera dedier aux utilisateurs avec le dhcp active, 
- le port 2 sera dedier aux serveurs sans dhcp 
- le port 3 sera dedier aux AP avec le dhcp active et le portail captif
- et la liaison l2t.root va nous servir de wan.

![forti interfaces](/assets/img/forti-interfaces.png)

Nous allons donc creer les regles de parefeu. Etant limite a 3 regles nous allons donc creer **FW-01, FW-05 et FW-07** conformement a notre tableau.

![forti-Firewall-policy](/assets/img/forti-Firewall-policy1.png)
![forti-Firewall-policy](/assets/img/forti-Firewall-policy2.png)


### Mise en place d'un portail captif

Une autre partie intéressante du travail a été l'expérimentation d'un portail captif sur le VLAN AP .

Le principe est de faire passer les utilisateurs par une page de contrôle avant de leur permettre d'accéder a internet.

Lors des tests, j'ai notamment vérifié :

- l'attribution ou la configuration des adresses IP ;
- l'accès à la passerelle du VLAN AP ;
- la communication avec les autres réseaux ;
- les politiques FortiGate ;
- la visibilité du trafic dans les logs ;
- le comportement du portail captif.




## Compétences développées

Ce projet m'a permis de consolider plusieurs compétences :

- Administration réseau ;
- VLAN et segmentation réseau ;
- Routage inter-VLAN ;
- Administration FortiGate ;
- Politiques de sécurité firewall ;
- Contrôle des accès ;
- Analyse des logs ;
- Troubleshooting réseau ;
- Compréhension des flux réseau ;
- Principe du moindre privilège.


## Ce que j'en retiens

Cette expérience m'a particulièrement intéressé, car elle montre que la sécurité réseau ne consiste pas uniquement à bloquer du trafic.
Nous devons être capables de comprendre :
> le flux → la règle → le comportement attendu → le comportement observé → la cause → la correction.

Ce projet constitue pour moi une expérience concrète à l'intersection entre **réseau, administration système et cybersécurité**. Aussi développer une méthode de travail basée sur l'observation, l'analyse des flux et la recherche méthodique de la cause d'un problème.


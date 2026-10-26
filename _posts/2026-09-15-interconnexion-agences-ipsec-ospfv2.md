---
title: "Interconnexion sécurisée de six agences avec IKEv2, IPsec et OSPFv2"
date: 2026-09-15 00:00:00 +0000
categories: [Network Security2, VPN]
tags: [cisco, ipsec, ikev2, ospf, mpls, fortigate, vpn, network-security]
image:
  path: /assets/img/Cisco-logo.png
  alt: Logo Cisco 
---

## Contexte

Dans le cadre de mes activités professionnelles en environnement de microfinance, j'ai participé à la mise en place d'une architecture permettant d'interconnecter plusieurs agences avec le siège. L'un des objectifs principaux de l'interconnexion VPN est de permettre aux utilisateurs des agences d'accéder à la application métier hébergée sur les serveurs du siège.

L'environnement repose sur un réseau **MPLS fourni par un opérateur**, utilisé comme infrastructure de transport entre les différents sites. Afin de sécuriser les communications entre les agences et le siège, des tunnels **IPsec négociés avec IKEv2** sont mis en place entre les routeurs Cisco.
L'architecture retenue repose sur un modèle **Hub-and-Spoke** :
Six routeurs représentent les six agences ; le septième routeur R7 est situé au siège ; R7 constitue le point central de communication avec les agences. Il est interconnecté au FortiGate du siège par l'intermédiaire d'un switch. Le FortiGate assure la sortie Internet du siège et des agences. Le routage dynamique entre les routeurs est assuré par **OSPFv2**.

> **Note de confidentialité :** Les informations présentées dans ce projet sont volontairement anonymisées. Les adresses exactes, noms de sites, paramètres VPN et informations propres à l'infrastructure de l'organisation ne sont pas publiés.
{: .prompt-info }
## Objectifs

Le projet avait plusieurs objectifs :

- interconnecter les six agences avec le siège ;
- Permettre l'acces aux applications metiers
- utiliser le réseau MPLS de l'opérateur comme réseau de transport ;
- sécuriser les communications intersites avec IPsec ;
- utiliser IKEv2 pour la négociation et l'authentification des tunnels ;
- mettre en place un routage dynamique avec OSPFv2 ;
- permettre aux agences d'accéder à Internet via le FortiGate du siège ;
- centraliser la sortie Internet au niveau du siège ;
- disposer d'une architecture évolutive et administrable.

## Architecture générale

L'architecture peut être représentée de manière simplifiée :

Voici les ressources utilisee dans notre lab :
Routeur CISCO :
- Système d’exploitation : 
- Mémoire RAM : 1gb
- nombre : 7

Switch CISCO :
- Système d'exploitation : 
- Mémoire RAM : 1gb
- nombre : 1

R7 joue le rôle de point central pour les communications entre les agences et le réseau du siège.

**image architecture a mettre**

Il n'y a pas de VLAN spécifique sur les routeurs des agences dans cette architecture. Les routeurs assurent principalement l'interconnexion des réseaux de leurs sites avec le réseau MPLS et le siège.

## Architecture Hub-and-Spoke

Le modèle retenu est une architecture **Hub-and-Spoke**.
R7 constitue le **Hub**, tandis que les six routeurs des agences constituent les **Spokes**.

![Topologie Hub and spoke](/assets/img/Hub-Spoke-Topology.jpg)


Chaque agence communique avec le routeur central R7 à travers l'infrastructure MPLS.

Cette architecture permet de centraliser les échanges au niveau du siège et de simplifier l'administration des tunnels.

## Plan d'adressage

Les routeurs utilisent un plan d'adressage basé sur **175.52.30.XX**.

Pour des raisons evidantes, les valeurs exactes utilisées ne sont pas publiées.

| Équipement | Site     | Adresse représentative |
| ---------- | -------- | ---------------------- |
| R1         | Agence 1 | 175.52.30.x            |
| R2         | Agence 2 | 175.52.30.X            |
| R3         | Agence 3 | 175.52.30.X            |
| R4         | Agence 4 | 175.52.30.X            |
| R5         | Agence 5 | 175.52.30.X            |
| R6         | Agence 6 | 175.52.30.X            |
| R7         | Siège    | 175.52.30.X            |


## Rôle du réseau MPLS

Le réseau MPLS fourni par l'opérateur constitue le **réseau de transport** entre les différents sites.
Il permet aux routeurs Cisco des agences de communiquer avec R7 au siège.
Il est important de distinguer le rôle du MPLS de celui du VPN. Le MPLS fournit donc la connectivité nécessaire au transport, tandis que le VPN ajoute une couche de sécurisation des communications.

## Mise en place du VPN IKEv2 / IPsec

Afin de sécuriser les communications intersites, un tunnel IPsec est établi entre chaque routeur d'agence et R7.

Le principe général est :

```text
R1 ───────────┐
R2 ───────────┤
R3 ───────────┤
R4 ───────────┼──── R7
R5 ───────────┤
R6 ───────────┘
```

R7 constitue ainsi le point central des connexions VPN.

### IKEv2

IKEv2 est utilisé pour négocier les paramètres de sécurité et authentifier les équipements participant à l'établissement du VPN.

### IPsec

IPsec assure ensuite la protection des communications transportées dans les tunnels, notamment en matière de confidentialité et d'intégrité.
Les paramètres cryptographiques, clés et secrets utilisés dans l'environnement professionnel ne sont pas publiés.

## Routage avec OSPFv2

Le routage dynamique est assuré par **OSPFv2**.
L'objectif est de permettre aux routeurs d'apprendre dynamiquement les réseaux accessibles à travers l'infrastructure.
OSPF permet ainsi d'éviter de dépendre uniquement de routes statiques et facilite l'évolution de l'infrastructure.

## Interconnexion R7 - FortiGate

R7 est situé dans le même environnement physique que le FortiGate au siège.
Les deux équipements sont reliés par câble au moyen d'un switch.
R7 joue donc le rôle de passerelle entre le trafic provenant des agences et de l'infrastructure du siège.
Le FortiGate prend ensuite en charge la sortie vers Internet et le contrôle des flux selon les politiques de sécurité configurées.

## Accès à une application métier du siège

R7 assure l'interconnexion et le routage entre les agences et le siège. Cependant, il ne constitue pas le point de sortie final vers les ressources internes ou Internet. Les flux provenant des agences sont acheminés vers le FortiGate du siège, qui constitue le point central de contrôle et de filtrage.

Selon la destination, le FortiGate dirige ensuite le trafic vers Internet ou vers les serveurs internes hébergeant les applications métier.

Cette architecture permet de centraliser la sortie Internet et d'appliquer les politiques de sécurité du FortiGate aux flux  des agences.

## Matrice des flux

| Source        | Destination               | Chemin                                          | Technologie                        | Objectif                                              |
| ------------- | ------------------------- | ----------------------------------------------- | ---------------------------------- | ----------------------------------------------------- |
| Agences 1 à 6 | Siège / serveurs internes | Agence → MPLS → VPN → R7 → FortiGate            | IKEv2 / IPsec + OSPFv2             | Accès sécurisé aux applications métier                |
| Siège         | Agences 1 à 6             | Serveurs → FortiGate → R7 → VPN → Agence        | IKEv2 / IPsec + OSPFv2             | Communication avec les ressources agences             |
| Agences 1 à 6 | Internet                  | Agence → MPLS → VPN → R7 → FortiGate → Internet | IKEv2 / IPsec + OSPFv2 + FortiGate | Accès Internet centralisé                             |
| Internet      | Agences 1 à 6             | Internet → FortiGate → R7 → MPLS → Agence       | FortiGate + routage                | Contrôler et bloquer les accès entrants non autorisés |
| R7            | FortiGate                 | R7 → Switch → FortiGate                         | Routage                            | Acheminer les flux des agences vers le pare-feu       |
| FortiGate     | Serveurs internes         | FortiGate → Réseau serveurs                     | Routage + politiques firewall      | Contrôler l'accès aux applications métier             |
| FortiGate     | Internet                  | FortiGate → WAN → Internet                      | NAT + politiques firewall          | Fournir l'accès Internet aux agences et au siège      |

## Vérification du VPN

Après la configuration, l'état des tunnels VPN doit être contrôlé.

Sur les routeurs Cisco, plusieurs commandes peuvent être utilisées :

```text
show crypto ikev2 sa
show crypto ipsec sa
```

Ces commandes permettent notamment de vérifier l'état de la négociation IKEv2 et des associations IPsec ainsi que l'activité des tunnels.

## Vérification d'OSPF

L'état des voisinages OSPF peut être vérifié avec :

```text
show ip ospf neighbor
```

L'objectif est de confirmer que les voisinages attendus sont établis.

Les routes apprises peuvent ensuite être vérifiées avec :

```text
show ip route
```

## Tests de connectivité

Une fois les tunnels et le routage opérationnels, des tests de bout en bout permettent de vérifier le fonctionnement global.


Les tests permettent notamment de vérifier :

- la communication entre une agence et le siège;
- la disponibilité des réseaux distants;
- le fonctionnement du routage OSPF;
- le passage du trafic vers le FortiGate;
- l'accès aux applications metiers;
- l'accès Internet depuis les agences.

## Méthodologie de troubleshooting

En cas de problème, l'analyse est effectuée progressivement afin d'identifier la couche responsable.
Cette méthode permet de distinguer un problème de transport, de VPN, de routage ou de filtrage.

Par exemple, une agence qui n'accède plus à Internet ne signifie pas nécessairement que le FortiGate est en cause. Le problème peut se situer au niveau du routage, du tunnel VPN, de la liaison MPLS ou de la communication entre R7 et le FortiGate.

## Sécurité de l'architecture

Plusieurs mécanismes contribuent à la sécurité :

- chiffrement des communications intersites avec IPsec ;
- négociation et authentification avec IKEv2 ;
- centralisation de la sortie Internet ;
- contrôle des flux par le FortiGate ;
- absence d'exposition directe des réseaux des agences à Internet ;
- routage dynamique avec OSPFv2 ;
- séparation entre réseau de transport MPLS et couche de sécurisation VPN.

Les clés, secrets, paramètres cryptographiques exacts et informations sensibles de l'infrastructure ne sont pas publiés.

## Compétences développées

Ce projet m'a permis de consolider plusieurs compétences :

- Administration de routeurs Cisco ;
- Routage dynamique OSPFv2 ;
- VPN site-à-site ;
- IKEv2 ;
- IPsec ;
- Compréhension des réseaux MPLS ;
- Architecture Hub-and-Spoke ;
- Interconnexion multi-sites ;
- Administration FortiGate ;
- Analyse des flux réseau ;
- Troubleshooting réseau ;
- Sécurisation des communications intersites.


## Conclusion

Ce projet constitue une extension du travail réalisé sur la segmentation et le contrôle des flux avec FortiGate.

Le premier projet était principalement orienté vers la sécurité du réseau local et le contrôle des communications entre différents segments.

Cette seconde architecture ajoute une dimension **multi-sites**, avec six agences interconnectées au siège par une infrastructure MPLS et des tunnels VPN IKEv2/IPsec, tandis que le routage dynamique est assuré par OSPFv2.

Le routeur R7 constitue le point central entre les agences et le siège, puis le trafic Internet des agences est acheminé vers le FortiGate avant de sortir vers Internet.

Cette expérience m'a permis de renforcer mes connaissances en réseau, VPN, routage et sécurité, tout en développant une méthodologie de troubleshooting applicable à des environnements d'infrastructure plus complexes.

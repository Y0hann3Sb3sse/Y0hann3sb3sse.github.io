---
title: "Mise en place d'un domaine Active Directory sécurisé sous Windows Server 2019"
date: 2026-09-07 10:00:00 +0000
categories: [Administration Systèmes, Sécurité Windows]
tags: [active-directory, windows-server, gpo, wsus, sécurité, infrastructure]
image:
  path: /assets/img/MS-AD.png
  alt: Logo Microsoft AD 
---

## Contexte

Projet réalisé dans le cadre de ma formation, avec pour but de reproduire une infrastructure Windows d'entreprise typique : un domaine Active Directory administré, avec des politiques de sécurité appliquées de façon centralisée et une gestion des mises à jour maîtrisée. L'objectif était de couvrir l'ensemble du cycle : de l'installation du contrôleur de domaine jusqu'à la vérification que les politiques appliquées sont réellement respectées sur le terrain.

> **Périmètre pédagogique :** l'organisation et les données décrites dans ce projet sont fictives.
{: .prompt-info }
## Objectifs

- Déployer un contrôleur de domaine Active Directory fonctionnel sous Windows Server 2019
- Structurer les groupes de sécurité selon une logique cohérente par service
- Centraliser la sécurisation et la gestion des postes clients via les GPO 
- Mettre en place une gestion centralisée des mises à jour Windows avec WSUS
- Vérifier que les politiques appliquées sont réellement effectives plutôt que de supposer qu'elles le sont

## Environnement

| Élément         | Détail                                   |
| --------------- | ---------------------------------------- |
| Système serveur | Windows Server 2019                      |
| Rôle principal  | Active Directory Domain Services (AD DS) |
| Rôle secondaire | WSUS (Windows Server Update Services)    |
| Postes clients  | Windows 10/11                            |
| Outils de test  | gpresult, rsop.msc                       |
| hyperviseur     | VMware workstation pro                   |


## Mise en œuvre

### 1. Installation et promotion du contrôleur de domaine

Ajout du rôle **AD DS** via le Gestionnaire de serveur, puis promotion du serveur en contrôleur de domaine pour créer un nouveau domaine. Points de vigilance à cette étape :
- Choix cohérent du nom de domaine (interne, non résolvable publiquement)
- Configuration du DNS intégré à l'Active Directory

### 2. Structuration des groupes de sécurité

Organisation des utilisateurs par **unités d'organisation (OU)** reflétant la structure de l'entreprise simulée ici par service comme comptabilite, team IT, direction ou encire service RH. Ensuite création de groupes de sécurité pour gérer les droits par fonction plutôt qu'individuellement (logique de moindre privilège : chaque utilisateur n'a accès qu'à ce dont il a besoin)



### 3. Gestion des GPO (Group Policy Objects)

Mise en place de stratégies de groupe pour sécuriser les postes clients sans intervention manuelle machine par machine. Exemples de politiques typiques dans ce genre de déploiement :
- Politique de mots de passe complexe et a durée de vie
- Restriction des droits d'installation de logiciels pour les utilisateurs standards
- Restriction des sur les disques amovibles 
- Fonds d'ecrans centralise pour harmoniser le parc informatique
- Ajouts des dossiers et disques reseaux directement dans l'emplacement des disques de la machine pour faciliter l'acces



### 4. Installation et configuration de WSUS

Déploiement de WSUS pour centraliser la distribution des mises à jour Windows plutôt que de laisser chaque poste se mettre à jour indépendamment. Cela permet de :
- Approuver les mises à jour avant déploiement (éviter qu'un correctif instable ne casse la production)
- Grouper les postes clients pour un déploiement progressif
- Avoir une visibilité centralisée sur l'état de mise à jour du parc

## Tests de conformité

Étape souvent négligée mais essentielle : vérifier que ce qui a été configuré est **réellement appliqué** sur les postes clients, et pas seulement correctement défini côté serveur.

- Utilisation de `gpresult /r` et `rsop.msc` sur les postes clients pour confirmer les GPO effectivement reçues
- Vérification de l'appartenance aux groupes de sécurité attendue
- Contrôle de l'état de mise à jour des postes via la console WSUS


## Compétences démontrées

- Administration Active Directory 
- Sécurisation d'un parc de postes via les stratégies de groupe
- Gestion centralisée des correctifs de sécurité 
- Démarche de vérification, pas seulement de déploiement

## Conclusion et pistes d'amélioration

En définitive, la centralisation de la gestion du parc informatique via Active Directory ne se limite pas à une simple harmonisation esthétique ou à une contrainte technique.Ce projet permet aussi de réduire drastiquement la surface d'attaque expose aux cybermenaces, tout en garantissant la conformité de notre infrastructure. Au-delà des gains immédiats en sécurité, l'implémentation d'Active Directory apporte une réponse concrète aux enjeux de productivité de l'entreprise.

---

*Projet réalisé dans le cadre de ma formation. Environnement de test isolé, aucune donnée réelle utilisée.*

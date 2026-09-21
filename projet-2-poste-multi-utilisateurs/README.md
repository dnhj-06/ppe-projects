---
title: PPE Projet 2 — Poste multi-utilisateurs pour salle informatique

---

---
title: PPE Projet 2 — Poste multi-utilisateurs pour salle informatique

---

# Rapport technique — Poste multi-utilisateurs de salle informatique

**Projet 2 — Classe E1B — Niveau Intermédiaire**



## Contexte

Un centre de formation équipe une salle informatique partagée par plusieurs étudiants qui se relaient dans la journée. Chacun doit disposer de son propre espace protégé, sans pouvoir accéder à celui des autres, et le poste doit se verrouiller seul entre deux utilisateurs.

| Élément | Détail |
|---|---|
| Machine | VM VMware Workstation nommée `PC-SALLE-01` |
| Nom Windows | `DESKTOP-FVNV7NS` (nom par défaut, le renommage n'est pas demandé par la consigne) |
| OS | Windows 10 Professionnel (64 bits) |
| Ressources | 4 Go RAM, 2 processeurs, disque 62 Go, carte réseau en mode Bridged |
| Gestion des comptes | 100 % en local, sans Active Directory (un seul poste, pas de domaine) |

---

## 1. Installation du système d'exploitation

Installation depuis l'ISO Windows 10 Pro fourni. Langue d'installation Français, format horaire et clavier **Français (Suisse)**. Pendant la configuration initiale (OOBE), l'option **"Configurer pour une organisation"** a été retenue puis contournée via "Rejoindre le domaine à la place", afin de créer un **compte local** et non un compte Microsoft — cohérent avec un poste de salle géré en autonomie.

🕐 14.09.2026 14:46:51
![01_assistant_vm_iso_detecte](https://hackmd.io/_uploads/HkUUW5LFGe.png)
 assistant de création de VM, ISO monté : **"Windows 10 x64 detected"**

🕐 14.09.2026 14:50:41
![02_nom_vm_pc_salle_01](https://hackmd.io/_uploads/Hkxvb98KMx.png)
 VM nommée **PC-SALLE-01** dans VMware Workstation

🕐 14.09.2026 14:53:57
![03_installation_langue_clavier_suisse](https://hackmd.io/_uploads/Skxd-9IKGe.png)
 installation Windows : langue Français, format et clavier **Français (Suisse)**

🕐 14.09.2026 15:39:13
![04_oobe_config_organisation](https://hackmd.io/_uploads/Hy2O-cIKze.png)
 configuration initiale, choix **"Configurer pour une organisation"**

🕐 14.09.2026 15:39:43
![05_oobe_nom_admin_maintenance](https://hackmd.io/_uploads/BJOtZ9LFGg.png)
 création du premier compte local : **`admin-maintenance`**

> ⚠️ **Point de vigilance :** Windows n'est pas activé (environnement de test/VM sans licence). À traiter avant toute mise en production réelle.

---

## 2. Compte administrateur dédié à la maintenance

Le premier compte créé pendant l'installation sert de **compte de maintenance**, volontairement nommé `admin-maintenance` pour le distinguer sans ambiguïté des comptes utilisateurs. C'est le seul compte disposant de privilèges d'administration sur le poste, conformément à l'objectif "séparer le compte de maintenance des comptes utilisateurs".

🕐 14.09.2026 15:57:10
![06_compte_admin_maintenance_confirme](https://hackmd.io/_uploads/Byqqb98KGe.png)
 : **Compte local**, **Administrateur** ✅

---

## 3. Comptes utilisateurs standards

Trois comptes locaux standards ont été créés pour les étudiants. Ils sont créés **sans compte Microsoft** et restent en type "standard" par défaut : aucun d'eux n'appartient au groupe Administrateurs.

### Architecture des comptes

| Compte | Type | Groupe | Rôle |
|---|---|---|---|
| `admin-maintenance` | Local | Administrateurs | Maintenance et configuration du poste |
| `Etudiant 1` | Local | Utilisateurs | Poste étudiant, espace personnel cloisonné |
| `Etudiant 2` | Local | Utilisateurs | Poste étudiant, espace personnel cloisonné |
| `Etudiant 3` | Local | Utilisateurs | Poste étudiant, espace personnel cloisonné |
| `Administrateur` | Intégré Windows | Administrateurs | Désactivé par défaut, non utilisé |
| `Invité`, `DefaultAccount`, `WDAGUtilityAccount` | Intégrés Windows | — | Comptes système, désactivés, non utilisés |

Chaque compte étudiant dispose d'un mot de passe individuel distinct, conforme à la politique définie à l'étape 6 (les valeurs ne figurent pas dans ce document, conformément aux bonnes pratiques).

🕐 14.09.2026 16:20:44
![07_trois_comptes_etudiants_crees](https://hackmd.io/_uploads/HyPiWcLtGx.png)
 les trois comptes `Etudiant 1`, `Etudiant 2` et `Etudiant 3` en **Compte local** ✅

🕐 15.09.2026 09:12:09
![08_net_user_liste_comptes](https://hackmd.io/_uploads/SJV3W98Ffe.png)
`net user` : liste complète des comptes locaux, avec l'orthographe exacte utilisée ensuite dans les commandes de permissions

---

## 4. Cloisonnement des fichiers personnels

Le cloisonnement repose sur deux niveaux complémentaires.

### 4.1 Cloisonnement natif des profils

Windows protège nativement chaque profil utilisateur (`C:\Users\<compte>`) par des permissions NTFS : seuls le propriétaire, le groupe Administrateurs et le compte Système y ont accès. Ces profils ne sont créés sur le disque qu'à la **première connexion** de chaque compte — les trois étudiants se sont donc connectés une fois pour les générer.

🕐 15.09.2026 08:52:02
![09_profils_utilisateurs_crees](https://hackmd.io/_uploads/B1g6bcUFfe.png)
 les profils `admin-maintenance`, `Etudiant 1`, `Etudiant 2` et `Etudiant 3` présents dans `C:\Utilisateurs`

🕐 15.09.2026 08:53:14
![10_securite_profil_etudiant1_natif](https://hackmd.io/_uploads/SJ6TbcUKfx.png)
 onglet Sécurité du profil `Etudiant 1` : *"Vous devez disposer d'autorisations d'accès en lecture pour afficher les propriétés de cet objet"* — même un administrateur ne lit pas ces permissions sans élévation, preuve du cloisonnement natif

### 4.2 Arborescence de travail cloisonnée

Une arborescence dédiée à la salle a été créée avec un dossier privé par étudiant et un dossier commun partagé, afin de démontrer une configuration explicite des permissions (et pas seulement le comportement par défaut de Windows).

| Dossier | Permissions appliquées |
|---|---|
| `C:\Salle-Info\Etudiant1` | `Etudiant 1` (Contrôle total), Administrateurs (Contrôle total), Système (Contrôle total) — héritage coupé |
| `C:\Salle-Info\Etudiant2` | `Etudiant 2` (Contrôle total), Administrateurs, Système — héritage coupé |
| `C:\Salle-Info\Etudiant3` | `Etudiant 3` (Contrôle total), Administrateurs, Système — héritage coupé |
| `C:\Salle-Info\Commun` | Groupe `Utilisateurs` (Modification), Administrateurs, Système — héritage coupé |

Les permissions ont été posées par script, avec `icacls` :

```
@echo off
icacls "C:\Salle-Info\Etudiant1" /inheritance:r /grant "Etudiant 1:(OI)(CI)F" /grant *S-1-5-32-544:(OI)(CI)F /grant *S-1-5-18:(OI)(CI)F
icacls "C:\Salle-Info\Etudiant2" /inheritance:r /grant "Etudiant 2:(OI)(CI)F" /grant *S-1-5-32-544:(OI)(CI)F /grant *S-1-5-18:(OI)(CI)F
icacls "C:\Salle-Info\Etudiant3" /inheritance:r /grant "Etudiant 3:(OI)(CI)F" /grant *S-1-5-32-544:(OI)(CI)F /grant *S-1-5-18:(OI)(CI)F
icacls "C:\Salle-Info\Commun" /inheritance:r /grant *S-1-5-32-545:(OI)(CI)M /grant *S-1-5-32-544:(OI)(CI)F /grant *S-1-5-18:(OI)(CI)F
pause
```

Explication des options : `/inheritance:r` coupe l'héritage des permissions du dossier parent (sans quoi le groupe Utilisateurs hériterait d'un accès et le cloisonnement serait inopérant), `(OI)(CI)` propage la règle aux fichiers et sous-dossiers, `F` accorde le contrôle total et `M` le droit de modification. Les identifiants `*S-1-5-32-544` (Administrateurs), `*S-1-5-32-545` (Utilisateurs) et `*S-1-5-18` (Système) sont les **SID** des groupes intégrés : ils sont utilisés à la place des noms parce que ceux-ci varient selon la langue de Windows, alors que les SID sont universels.

Passer par un script plutôt que par des commandes isolées présente un autre avantage : la configuration est reproductible à l'identique sur d'autres postes de la salle.

🕐 15.09.2026 09:16:30
![11_script_permissions_blocnotes](https://hackmd.io/_uploads/rkzyG5UYfl.png)
 le script `permissions.bat` avant enregistrement

🕐 15.09.2026 09:20:53
![12_arborescence_salle_info](https://hackmd.io/_uploads/H1q1GcLYfx.png)
 arborescence `C:\Salle-Info` : un dossier par étudiant, un dossier `Commun`, et le script

🕐 15.09.2026 09:21:17
![13_execution_script_permissions](https://hackmd.io/_uploads/SyXlf9IYfe.png)
 exécution du script : les quatre dossiers traités sans erreur ✅

🕐 15.09.2026 09:23:54
![14_verification_icacls_etudiant1_commun](https://hackmd.io/_uploads/HyQbz5LFMe.png)
 vérification : `Etudiant1` n'expose que trois entrées (Système, Administrateurs, `Etudiant 1`) tandis que `Commun` ajoute le groupe `Utilisateurs` en `(M)` ✅

---

## 5. Verrouillage automatique après inactivité

Le verrouillage automatique est indispensable ici : sans lui, un étudiant qui quitte le poste sans fermer sa session laisserait le suivant travailler dans sa session, ce qui annulerait tout le cloisonnement mis en place à l'étape 4.

Le paramètre utilisé est **Stratégies locales > Options de sécurité > "Ouverture de session interactive : limite d'inactivité de l'ordinateur"** dans `secpol.msc`.

🕐 15.09.2026 09:26:16
![15_limite_inactivite_non_definie](https://hackmd.io/_uploads/B1kzzcIYfx.png)
 état initial : le paramètre est sur **"Non défini"**

🕐 15.09.2026 09:30:09
![16_mauvais_parametre_seuil_verrouillage](https://hackmd.io/_uploads/H1AMM5IYGl.png)
premier essai sur le mauvais paramètre : **"seuil de verrouillage du compte d'ordinateur"**, qui compte des *tentatives de connexion non valides* et n'a rien à voir avec l'inactivité

> **Incident :** après ce premier réglage, aucun verrouillage ne se déclenchait. **Cause :** le paramètre modifié était le seuil de verrouillage de compte (nombre de tentatives de connexion ratées) et non la limite d'inactivité — deux lignes voisines dans la liste, aux libellés proches. **Résolution :** vérification en base de registre pour lever le doute, puis réglage du bon paramètre et revalidation.

🕐 15.09.2026 09:34:01
![17_reg_query_valeur_absente](https://hackmd.io/_uploads/S1JNG5IYfl.png)
 `reg query` sur `InactivityTimeoutSecs` : la valeur n'existe pas, confirmation que la stratégie d'inactivité n'était pas appliquée

🕐 15.09.2026 09:35:05
![18_parametre_limite_inactivite_60s](https://hackmd.io/_uploads/rkqBz5IFfe.png)
 — cette fois le bon paramètre : **"limite d'inactivité de l'ordinateur"**, réglé sur 60 secondes pour le test

🕐 15.09.2026 09:37:34
![19_limite_inactivite_60_secondes](https://hackmd.io/_uploads/ByHIGc8KMe.png)
 la stratégie affiche désormais **60 secondes**

🕐 15.09.2026 09:38:48
![20_reg_query_0x3c](https://hackmd.io/_uploads/rkXPfqUFMg.png)
`reg query` renvoie `InactivityTimeoutSecs REG_DWORD 0x3c` (60 en décimal) : la stratégie est bien écrite en registre ✅

🕐 15.09.2026 09:48:24
![image](https://hackmd.io/_uploads/rJI_m5UYGl.png)

 le poste s'est verrouillé seul après la minute d'inactivité ✅

🕐 15.09.2026 09:51:13
![22_limite_inactivite_300_secondes](https://hackmd.io/_uploads/HkD_zqLYMx.png)
 valeur définitive retenue : **300 secondes**

**Justification de la valeur :** 5 minutes est un compromis adapté à une salle où les étudiants se relaient — assez court pour qu'un poste abandonné se verrouille avant qu'un autre étudiant s'y installe, assez long pour ne pas verrouiller quelqu'un en train de lire un énoncé.

---

## 6. Politique de mot de passe

Configurée dans `secpol.msc` → **Stratégies de comptes > Stratégie de mot de passe**.

| Paramètre | Valeur par défaut | Valeur appliquée | Justification |
|---|---|---|---|
| Longueur minimale | 0 caractère | **8 caractères** | Minimum raisonnable pour un compte étudiant |
| Exigences de complexité | Désactivé | **Activé** | Impose majuscules, minuscules et chiffres |
| Durée de vie maximale | 42 jours | **90 jours** | Rotation régulière sans être pénible en contexte scolaire |
| Durée de vie minimale | 0 jour | **1 jour** | Empêche de changer plusieurs fois d'affilée pour revenir à l'ancien mot de passe |
| Historique des mots de passe | 0 | **5 mémorisés** | Empêche la réutilisation immédiate |
| Chiffrement réversible | Désactivé | **Désactivé** (inchangé) | Activer cette option reviendrait à stocker les mots de passe en clair |

🕐 15.09.2026 09:52:35
![23_strategie_mot_de_passe_avant](https://hackmd.io/_uploads/Sy6FM9UYMg.png)
 état initial de la stratégie

🕐 15.09.2026 09:56:31
![24_strategie_mot_de_passe_apres](https://hackmd.io/_uploads/rk_cz98tMe.png)
 stratégie configurée : historique 5, 90 jours, 1 jour, complexité activée, longueur minimale 8 ✅

### Vérification de l'application réelle

🕐 15.09.2026 09:59:45
![25_net_user_contourne_politique](https://hackmd.io/_uploads/HJKjf5Ltfx.png)
`net user "Etudiant 1" abc` exécuté en administrateur : **la commande réussit** malgré la politique

> **Incident :** ce premier test laissait croire que la politique n'était pas appliquée. **Cause :** `net user` lancé par un administrateur passe par une API de *réinitialisation* de mot de passe (`NetUserSetInfo`, niveau 1003) qui n'applique pas les contrôles de longueur et de complexité — ceux-ci ne s'imposent qu'aux changements effectués par l'utilisateur lui-même ou via l'interface Windows. **Résolution :** test refait par le chemin normal (création de compte via l'interface), et le mot de passe faible posé pendant le test a immédiatement été remplacé par un mot de passe conforme.

🕐 15.09.2026 10:29:02
![26_refus_mot_de_passe_faible_interface](https://hackmd.io/_uploads/Syrhz5LKzl.png)
 test par l'interface : *"Le mot de passe que vous avez entré ne répond pas aux exigences de complexité définies par l'administrateur"* ✅

**Remarque technique :** la règle de complexité de Windows refuse également tout mot de passe contenant le nom du compte. Avec des comptes nommés `Etudiant 1/2/3`, un mot de passe du type `Etudiant2026` est donc rejeté lors d'un changement par l'utilisateur. Les mots de passe attribués ont été choisis en conséquence, sans reprendre le nom du compte.

---

## 7. Test du cloisonnement avec chaque compte

Chaque compte étudiant a été testé en conditions réelles : accès à son propre espace, tentative d'accès à l'espace d'un autre, accès au dossier commun.

### Etudiant 1

🕐 15.09.2026 10:42:15
![27_etudiant1_acces_son_dossier](https://hackmd.io/_uploads/B1bTf9LtMl.png)
`Etudiant 1` accède à `C:\Salle-Info\Etudiant1` et y crée son fichier personnel ✅

🕐 15.09.2026 10:43:22
![28_etudiant1_refus_dossier_etudiant2](https://hackmd.io/_uploads/r1spGqIFMl.png)
 tentative d'ouverture de `C:\Salle-Info\Etudiant2` depuis l'Explorateur : *"Vous ne disposez pas des autorisations requises pour accéder à ce dossier"* ❌

🕐 15.09.2026 10:45:06
![29_etudiant1_whoami_groupe_administrateurs](https://hackmd.io/_uploads/Sk_Az9LYGe.png)
`whoami` confirme la session `etudiant 1`, et `net localgroup Administrateurs` montre que le groupe ne contient que `Administrateur` (intégré, désactivé) et `admin-maintenance` : aucun compte étudiant n'est administrateur ✅

🕐 15.09.2026 10:57:07
![30_dir_fichier_introuvable_trompeur](https://hackmd.io/_uploads/Hyrk7qLtMe.png)
 `dir Etudiant2` renvoie **"Fichier introuvable"** au lieu d'un refus explicite

> **Incident :** cette réponse laissait penser que le dossier était accessible mais vide. **Cause :** lorsque `dir` n'a pas le droit d'énumérer un dossier, cmd remonte parfois un code d'erreur traduit par "Fichier introuvable" plutôt que "Accès refusé" — un faux négatif classique. **Résolution :** vérification avec `icacls` et `cd`, qui renvoient tous deux un refus explicite.

🕐 15.09.2026 11:04:41
![31_etudiant1_icacls_acces_refuse](https://hackmd.io/_uploads/r1elm5IYGe.png)
 `icacls Etudiant2` depuis la session `etudiant 1` : **"Accès refusé"** — `Etudiant 1` ne peut même pas lire les permissions du dossier d'un autre ❌

### Etudiant 2

🕐 15.09.2026 11:09:39
![32_etudiant2_serie_verification](https://hackmd.io/_uploads/Hknl79IKMg.png)
 `whoami` confirme `etudiant 2`, `dir Etudiant2` liste son propre fichier personnel, `dir Commun` donne accès au dossier partagé ✅

🕐 15.09.2026 11:11:57
![33_etudiant2_cd_acces_refuse](https://hackmd.io/_uploads/BJI-X5IYzx.png)
 `cd Etudiant1` : **"Accès refusé"** ❌

🕐 15.09.2026 11:13:30
![34_etudiant2_acces_son_dossier](https://hackmd.io/_uploads/Sknf79ItGl.png)
accès à son propre dossier depuis l'Explorateur, avec son fichier personnel ✅

### Etudiant 3

🕐 15.09.2026 11:20:22
![35_etudiant3_serie_complete](https://hackmd.io/_uploads/HJF7mc8YGg.png)
 capture de synthèse : `whoami` confirme `etudiant 3`, `cd Etudiant1` est **refusé** ❌, `dir Etudiant3` liste son propre fichier ✅, et `dir Commun` affiche `note-etudiant2.txt` déposé par `Etudiant 2` ✅ — le partage fonctionne dans l'espace commun alors que les espaces privés restent hermétiques

### Synthèse des tests

| Compte testé | Son propre dossier | Dossier d'un autre étudiant | Dossier commun |
|---|---|---|---|
| `Etudiant 1` | Accès ✅ | Refusé ❌ | Accès ✅ |
| `Etudiant 2` | Accès ✅ | Refusé ❌ | Accès ✅ |
| `Etudiant 3` | Accès ✅ | Refusé ❌ | Accès ✅ |

---

## 8. Notes et incidents rencontrés

| Incident | Cause | Résolution |
|---|---|---|
| La limite d'inactivité restait "Non défini" et aucun verrouillage ne se déclenchait | Le paramètre modifié était "seuil de verrouillage du compte d'ordinateur" (tentatives de connexion ratées) et non "limite d'inactivité de l'ordinateur" — libellés voisins dans la même liste | Contrôle en base de registre (`InactivityTimeoutSecs` absent), puis réglage du bon paramètre et revalidation en registre (`0x3c`) |
| `net user "Etudiant 1" abc` accepté malgré la politique de mot de passe | Une réinitialisation lancée par un administrateur passe par une API qui n'applique pas les contrôles de longueur et de complexité | Test refait via l'interface de création de compte, qui applique bien la politique et refuse le mot de passe faible ; mot de passe de test aussitôt remplacé par un mot de passe conforme |
| `dir` sur le dossier d'un autre étudiant renvoie "Fichier introuvable" au lieu de "Accès refusé" | Traduction trompeuse du code d'erreur par cmd lorsque l'énumération d'un dossier est refusée | Diagnostic confirmé avec `icacls` et `cd`, qui renvoient tous deux "Accès refusé" |

## Points de vigilance

- **Windows non activé** : environnement de test sans licence. Une licence Windows 10 Pro devra être appliquée avant mise en production réelle.
- **Nom Windows du poste** : la VM est nommée `PC-SALLE-01` côté VMware, mais le nom Windows est resté `DESKTOP-FVNV7NS`. Le renommage n'est pas demandé par la consigne, mais serait à faire dans un déploiement réel pour identifier le poste sur le réseau de l'établissement.
- **Mot de passe et nom de compte** : la règle de complexité interdisant les mots de passe contenant le nom du compte, une convention de nommage des mots de passe indépendante des noms d'utilisateurs doit être retenue pour les futurs comptes.

## Conclusion

Le poste est opérationnel et conforme aux objectifs du projet : un compte de maintenance administrateur clairement séparé des trois comptes étudiants standards, des espaces personnels cloisonnés à deux niveaux (profils Windows natifs et arborescence `C:\Salle-Info` avec permissions NTFS explicites posées par script), un dossier commun partagé pour les échanges, un verrouillage automatique après 5 minutes d'inactivité, et une politique de mot de passe appliquée et vérifiée. Le cloisonnement a été testé avec chacun des trois comptes : chaque étudiant accède à son espace et au dossier commun, et se voit refuser l'accès aux espaces des autres.
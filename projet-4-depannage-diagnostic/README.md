---
title: PPE Projet 4 — Dépannage et diagnostic d'un poste en panne

---

---
title: PPE Projet 4 — Dépannage et diagnostic d'un poste en panne
---

# Rapport d'intervention — Ticket #PPE4-2026

**Projet 4 — Classe E1B — Niveau Intermédiaire**

---

## Fiche du ticket

| Champ | Valeur |
|---|---|
| N° de ticket | PPE4-2026 |
| Poste concerné | `PC-SUPPORT-01` (VM VMware Workstation, clone de `PC-SALLE-01`) |
| OS | Windows 10 Professionnel (64 bits) |
| Compte technicien | `admin-maintenance` (administrateur local) |
| Symptôme signalé | « Plus aucun accès réseau ni Internet » |
| Catégorie | Matériel / pilote de périphérique |
| Sévérité | Haute (poste isolé du réseau) |
| Statut final | ✅ Résolu — poste fonctionnel |

**Périmètre de la simulation :** conformément aux consignes du projet, la panne a été **provoquée volontairement** sur un poste de test dédié, afin de dérouler une véritable démarche de diagnostic. Le poste de test est un **clone** de `PC-SALLE-01` (Projet 2), choisi parce qu'il est en adressage DHCP : les postes des projets précédents restent ainsi intacts et aucun conflit d'adresse IP fixe n'est possible.

---

## 1. Préparation du poste de test

Avant toute intervention, un poste de test a été isolé et son **état sain a été relevé**. Ce point de référence « avant » est indispensable : sans lui, impossible de prouver ensuite que le poste a réellement été réparé.

🕐 16.09.2026 13:49:54
![01_clonage_pc_salle_01](https://hackmd.io/_uploads/BJgB6GdFfx.png)
clonage complet de `PC-SALLE-01` dans VMware Workstation (full clone), renommé `PC-SUPPORT-01`. Les postes des projets 1 à 3 restent visibles et intacts dans la bibliothèque.

### État sain de référence (avant panne)

🕐 16.09.2026 14:09:37
![02_etat_sain_carte_reseau](https://hackmd.io/_uploads/S1hH6GuYfe.png)
 Gestionnaire de périphériques : la carte **Intel(R) 82574L Gigabit Network Connection** affiche « **Ce périphérique fonctionne correctement** » ✅

🕐 16.09.2026 14:10:42
![03_etat_sain_ipconfig](https://hackmd.io/_uploads/SJU8azOFGl.png)
`ipconfig` : la carte Ethernet0 possède l'adresse **192.168.20.178**, masque /24, passerelle 192.168.20.1

🕐 16.09.2026 14:11:25
![04_etat_sain_navigation](https://hackmd.io/_uploads/ByQwaGuKfg.png)
 la navigation Internet fonctionne (site de l'école chargé)

🕐 16.09.2026 14:13:28
![05_etat_sain_ping](https://hackmd.io/_uploads/B1V_TMutfl.png)
`ping 8.8.8.8` : **4 paquets envoyés, 4 reçus, 0 % de perte** ✅

🕐 16.09.2026 14:18:42
![06_pilote_origine_reference](https://hackmd.io/_uploads/B1kt6fdFfx.png)
 pilote d'origine relevé pour référence : fournisseur **Microsoft**, version **12.17.10.8**, date **12.06.2018**. Ce sera le pilote « sain » à retrouver après réparation.

---

## 2. Reproduction de la panne

La panne retenue est une **panne de pilote de carte réseau**, l'un des scénarios proposés par le formateur. Elle a été provoquée en forçant l'installation d'un **pilote incompatible** sur la carte Ethernet — méthode qui reproduit fidèlement un pilote corrompu ou mal installé rencontré en production.

🕐 16.09.2026 14:20:23
![07_mise_a_jour_choisir_liste](https://hackmd.io/_uploads/By9tpzdKfg.png)
 Mettre à jour le pilote → « Choisir parmi une liste de pilotes disponibles sur mon ordinateur »

🕐 16.09.2026 14:23:11
![08_liste_pilotes_incompatibles](https://hackmd.io/_uploads/HJLcTf_Ffl.png)
 la case **« Afficher les matériels compatibles » est décochée** : on accède ainsi à des pilotes qui ne correspondent pas à la carte

🕐 16.09.2026 14:24:15
![09_avertissement_pilote_ignore](https://hackmd.io/_uploads/Hyljpz_tGl.png)
 Windows avertit que le pilote n'est pas garanti compatible et que le matériel « peut cesser complètement de fonctionner ». Avertissement volontairement ignoré pour créer la panne.

🕐 16.09.2026 14:25:53
![10_installation_pilote_echec_code10](https://hackmd.io/_uploads/HkOj6GdKGe.png)
 le pilote **Intel(R) Centrino(R) Wireless-N 135** (un pilote de carte **Wi-Fi**, posé sur une carte **filaire**) est installé mais refuse de démarrer : **« Ce périphérique ne peut pas démarrer. (Code 10) »**

---

## 3. Constat des symptômes (prise en charge du ticket)

À partir d'ici, l'intervention est menée comme un vrai ticket : le technicien constate les symptômes sans présumer de la cause.

🕐 16.09.2026 14:27:12
![11_triangle_jaune_carte](https://hackmd.io/_uploads/S1Bh6fdtGg.png)
un **triangle jaune** ⚠️ apparaît sur la carte réseau, devenue « Intel(R) Centrino(R) Wireless-N 135 »

🕐 16.09.2026 14:27:32
![12_code10_detail](https://hackmd.io/_uploads/HykTTzuKMe.png)
 détail de l'état du périphérique : **Code 10 — « L'opération a échoué / L'opération demandée n'a pas pu être menée à bien »**

🕐 16.09.2026 14:28:03
![13_symptome_reseau_absent](https://hackmd.io/_uploads/S1M40zOKfg.png)
 côté utilisateur : `ipconfig` ne montre plus aucune adresse pour Ethernet0 (**média déconnecté**) et `ping 8.8.8.8` échoue en **« Défaillance générale »**

---

## 4. Démarche de diagnostic

Le diagnostic suit une logique **du plus simple au plus profond** (des couches basses vers les couches hautes), en documentant chaque hypothèse, **y compris celles qui sont écartées**.

### H1 — Liaison physique (le « câble »)

Premier réflexe support : vérifier la connectique avant tout.

🕐 16.09.2026 14:35:09
![14_h1_liaison_physique](https://hackmd.io/_uploads/Sk2vRzdFMx.png)
 dans VMware, l'adaptateur réseau est **Connected** et **Connect at power on** est coché, en mode Bridged. Le lien physique est bon.
➡️ **Hypothèse écartée.**

### H2 — Carte désactivée dans Windows

🕐 16.09.2026 14:38:09
![15_h2_carte_absente_ncpa](https://hackmd.io/_uploads/rytu0MOtMe.png)
 dans `ncpa.cpl`, il ne reste que la connexion Bluetooth : la carte **Ethernet0 a complètement disparu** de la liste.
➡️ **Hypothèse écartée** — et indice majeur : une carte simplement désactivée resterait visible (grisée). Une interface qui **disparaît** signifie que Windows n'arrive pas à charger son pilote. Cela écarte du même coup une simple erreur de configuration IP : on ne peut pas configurer une carte qui ne se présente plus au système.

### H3 — Service réseau arrêté

🕐 16.09.2026 14:43:38
![16_h3_service_dhcp_actif](https://hackmd.io/_uploads/HywtCfOKfx.png)
 dans `services.msc`, le service **Client DHCP** est **En cours d'exécution** (démarrage Automatique).
➡️ **Hypothèse écartée** — les services réseau tournent, l'absence d'adresse ne vient pas de là.

### H4 — Pilote défectueux (cause probable)

Le faisceau d'indices pointe vers le pilote : triangle jaune, **Code 10**, carte absente de `ncpa.cpl`. Le Gestionnaire de périphériques (captures `11` et `12`) confirme un pilote qui ne peut pas démarrer.
➡️ **Cause identifiée : pilote incompatible chargé sur la carte réseau.**

### H5 — Confirmation par l'Observateur d'événements

Le diagnostic est corroboré par les journaux système, comme demandé dans les consignes.

🕐 16.09.2026 15:05:54
![17_journaux_systeme_erreurs](https://hackmd.io/_uploads/rJljRM_tGl.png)
journal **Système** : deux erreurs rouges (source **NETwNe64**, ID 5006 et 5001) à **14:25:49**, exactement l'heure de l'installation du mauvais pilote. Juste en dessous, un avertissement **e1i65x64** — le pilote de la vraie carte Intel 82574L qui se plaint en parallèle : les deux pilotes se marchent dessus.

🕐 16.09.2026 15:07:36
![18_journal_preuve_pilote_incorrect](https://hackmd.io/_uploads/SkioCzuKfg.png)
 détail de l'événement 5006 : **« Intel(R) Centrino(R) Wireless-N 135 : le numéro de version est incorrect pour ce pilote. »** La cause est prouvée noir sur blanc.

---

## 5. Résolution

La réparation applique le geste exactement inverse de la panne : **restaurer le pilote précédent**. Windows conserve en mémoire le pilote d'origine (celui relevé à la capture `06`), ce qui permet un retour arrière propre sans réinstallation manuelle.

🕐 16.09.2026 15:18:53
![19_restauration_pilote](https://hackmd.io/_uploads/BJS2CMdKGx.png)
onglet Pilote → **Restaurer le pilote** → motif « Mes applications ne fonctionnent pas avec ce pilote » → Oui

🕐 16.09.2026 15:19:17
![20_carte_restauree](https://hackmd.io/_uploads/Sk0nAMOtfx.png)
 la carte redevient **Intel(R) 82574L Gigabit Network Connection**, sans triangle jaune ✅

---

## 6. Vérification du rétablissement

Le retour à la normale est prouvé sur les trois mêmes plans que l'état sain de départ.

🕐 16.09.2026 15:22:47
![21_verif_carte_saine](https://hackmd.io/_uploads/ByPaCz_tfg.png)
 Gestionnaire de périphériques : « **Ce périphérique fonctionne correctement** » ✅ (comparaison directe avec le Code 10 précédent)

🕐 16.09.2026 15:23:13
![22_verif_reseau_retabli](https://hackmd.io/_uploads/HyzACzuKfl.png)
 `ipconfig` retrouve l'adresse **192.168.20.178** et `ping 8.8.8.8` répond **4/4, 0 % de perte** ✅

🕐 16.09.2026 15:23:46
![23_verif_navigation](https://hackmd.io/_uploads/rk9C0GuKMg.png)
 la navigation Internet fonctionne à nouveau ✅

---

## Résumé des hypothèses testées

| # | Hypothèse | Outil utilisé | Résultat |
|---|---|---|---|
| H1 | Liaison physique débranchée | Paramètres VMware | ❌ Écartée — adaptateur connecté |
| H2 | Carte désactivée dans Windows | `ncpa.cpl` | ❌ Écartée — carte absente (indice pilote) |
| H3 | Service réseau (DHCP) arrêté | `services.msc` | ❌ Écartée — service en cours d'exécution |
| H4 | Pilote défectueux | Gestionnaire de périphériques | ✅ **Confirmée** — Code 10 |
| H5 | Confirmation de la cause | Observateur d'événements | ✅ **Prouvée** — NETwNe64, ID 5006 |

**Cause racine :** un pilote incompatible (Intel Centrino Wireless-N 135, pilote Wi-Fi) chargé sur la carte Ethernet Intel 82574L, provoquant un échec de démarrage du périphérique (Code 10) et la disparition de l'interface réseau.

**Correctif appliqué :** restauration du pilote précédent (Restaurer le pilote), retour au pilote Microsoft d'origine 12.17.10.8.

---

## Points de vigilance

- **Diagnostic vs données :** une interface qui disparaît de `ncpa.cpl` peut faire croire à une carte débranchée ou retirée. Le Gestionnaire de périphériques (triangle jaune + code d'erreur) tranche entre un problème matériel/pilote et un problème de configuration.
- **Restaurer le pilote plutôt que réinstaller :** le bouton « Restaurer le pilote » n'est disponible que si un pilote précédent existe. C'est la méthode la plus rapide et la plus sûre après une mauvaise mise à jour ; en son absence, on désinstalle le périphérique (en supprimant le pilote) puis on lance « Rechercher les modifications matérielles ».
- **Code 10 :** signifie « le périphérique ne peut pas démarrer », souvent lié à un pilote incompatible ou corrompu — pas nécessairement à une panne matérielle réelle.
- **Corroborer par les journaux :** le message de l'Observateur d'événements (« le numéro de version est incorrect pour ce pilote ») confirme la cause de façon indépendante du Gestionnaire de périphériques, ce qui renforce la fiabilité du diagnostic.

## Conclusion

Le poste `PC-SUPPORT-01` a été diagnostiqué et réparé selon une démarche structurée : relevé de l'état sain, constat des symptômes, test méthodique de cinq hypothèses (quatre écartées avec preuve, une confirmée), corroboration par l'Observateur d'événements, puis correction et vérification du rétablissement sur les trois plans (matériel, configuration réseau, usage). La cause — un pilote réseau incompatible provoquant un Code 10 — a été identifiée, prouvée par les journaux, et corrigée par la restauration du pilote d'origine. Le poste est de nouveau pleinement fonctionnel.
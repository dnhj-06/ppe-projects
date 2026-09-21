---
title: PPE Projet 3 — Migration et sauvegarde d'un poste existant

---

---
title: PPE Projet 3 — Migration et sauvegarde d'un poste existant

---

# Procédure de migration — PC-COMPTA-01

**Projet 3 — Classe E1B — Niveau Intermédiaire**


## Contexte

Un poste vieillissant doit être réinstallé complètement, mais l'utilisatrice doit retrouver ses documents et ses paramètres après l'opération.

| Élément | Détail |
|---|---|
| Poste migré | `PC-COMPTA-01` (VM VMware Workstation, clone du poste du Projet 1) |
| Utilisatrice | Sophie Martin, comptable |
| OS avant et après | Windows 10 Professionnel (64 bits) |
| Support de sauvegarde | Disque externe simulé par un second disque virtuel de 10 Go (lecteur `E:` "Sauvegarde") |
| Données à migrer | Profil utilisateur de Sophie Martin (Documents, Bureau, Images, Téléchargements) |

**Choix de méthode :** la migration a été réalisée sur un **clone** du poste d'origine, afin de conserver intact le poste du Projet 1 qui doit encore être présenté. Le clone porte le même nom Windows que l'original, ce qui impose de ne jamais démarrer les deux VM simultanément (voir points de vigilance).

---

## Vue d'ensemble de la procédure

| Phase | Action | Outil utilisé |
|---|---|---|
| 1 | Inventaire des données et des paramètres | `inventaire.bat` (wmic, ipconfig, net user, reg query, dir) |
| 2 | Sauvegarde sur support externe | `sauvegarde.bat` (robocopy avec journal) |
| 3 | Vérification d'intégrité avant réinstallation | `verif-integrite.ps1` (empreintes SHA-256) |
| 4 | Réinstallation propre du système | Support d'installation Windows 10 Pro |
| 5 | Réinstallation des logiciels | Téléchargement depuis les sites éditeurs |
| 6 | Restauration et vérification d'intégrité | `restauration.bat` puis `verif-integrite.ps1` |
| 7 | Reconfiguration des paramètres | sysdm.cpl, Paramètres réseau, Imprimantes |
| 8 | Documentation | Ce document + la boîte à outils sur le support externe |

---

## 1. Inventaire des données à conserver

Un inventaire de migration ne se limite pas aux fichiers : il doit couvrir tout ce qui devra être reconstitué après la réinstallation, car un disque système formaté emporte aussi les comptes, la configuration réseau, les imprimantes et la liste des logiciels.

L'inventaire a été généré automatiquement dans un fichier texte, qui sert à la fois de preuve et de référence pendant la phase de reconfiguration.

**Éléments inventoriés :** nom de machine et groupe de travail, configuration réseau complète, comptes locaux, imprimantes installées, logiciels installés (via les clés de désinstallation du registre), et contenu des dossiers utilisateur.

🕐 15.09.2026 13:40:12
![01_creation_donnees_utilisateur](https://hackmd.io/_uploads/Bk0Lr1dKfl.png)
 création du jeu de données de travail dans le profil de Sophie Martin (factures, suivi TVA, dossier Clients, notes sur le Bureau)

🕐 15.09.2026 14:38:53
![02_inventaire_erreur_chemin_profil](https://hackmd.io/_uploads/H1ODB1uFMx.png)
premier passage du script : quatre erreurs "Le fichier spécifié est introuvable" sur les dossiers utilisateur

> **Incident :** l'inventaire s'exécutait sans erreur apparente mais ne contenait aucune donnée utilisateur. **Cause :** le script visait `C:\Users\s.martin`, alors que le dossier de profil créé par Windows porte le nom complet du compte, `C:\Users\Sophie Martin` — Windows nomme le dossier d'après le nom du compte, pas d'après une convention supposée. **Résolution :** vérification du nom réel avec `dir C:\Users`, puis correction du chemin dans le script.

🕐 15.09.2026 14:56:46
![03_dossier_profil_sophie_martin](https://hackmd.io/_uploads/rJhuSJuYGl.png)
`dir C:\Users` confirme le nom réel du dossier de profil

🕐 15.09.2026 14:59:56
![04_inventaire_donnees_utilisateur](https://hackmd.io/_uploads/SJrKHyuKGg.png)
 section "Données utilisateur" de l'inventaire après correction : le dossier `Clients`, `dossier compta.pdf`, `facture-client-septembre.txt` et `suivi-tva-2026.csv` ✅

---

## 2. Sauvegarde sur support externe

### 2.1 Préparation du support

Le support de sauvegarde est un second disque virtuel de 10 Go, qui joue le rôle d'un disque dur externe : il est indépendant du disque système et pourra être détaché pendant la réinstallation.

🕐 15.09.2026 15:07:54
![05_gestion_disques_avant_ajout](https://hackmd.io/_uploads/BkTYHkuFGx.png)
 état initial : seul le disque système est présent

🕐 15.09.2026 15:10:03
![06_vm_settings_un_seul_disque](https://hackmd.io/_uploads/Hkq4Ikutzl.png)
 côté VMware, un seul "Hard Disk" est déclaré : l'ajout du disque n'avait pas été enregistré

🕐 15.09.2026 15:11:17
![07_erreur_creation_disque_chemin](https://hackmd.io/_uploads/By-S8yuKGg.png)
nouvelle tentative : *"The path ... cannot be written to"*

> **Incident :** VMware refusait de créer le fichier du disque dans le dossier de la VM. **Cause :** le dossier `Documents\Virtual Machines` n'était pas inscriptible au moment de l'opération (dossier Documents synchronisé). **Résolution :** le fichier `disque-sauvegarde.vmdk` a été créé à un autre emplacement — le fichier de disque n'a pas besoin de résider dans le dossier de la VM, VMware ne conserve qu'une référence vers son chemin.

🕐 15.09.2026 15:14:39
![08_vm_settings_disque_ajoute](https://hackmd.io/_uploads/rJdSLyuKzg.png)
le second disque de 10 Go apparaît dans la configuration de la VM ✅

🕐 15.09.2026 15:17:04
![09_initialisation_disque_gpt](https://hackmd.io/_uploads/SykLIJuFGx.png)
 initialisation du disque en table de partition **GPT**

🕐 15.09.2026 15:18:22
![10_disque1_non_alloue](https://hackmd.io/_uploads/S1jSvydFfe.png)

 le Disque 1 est visible avec 9,98 Go non alloués

🕐 15.09.2026 15:25:22
![11_formatage_ntfs_sauvegarde](https://hackmd.io/_uploads/Sk5wDJOtfe.png)
 formatage en **NTFS**, nom de volume **Sauvegarde**

🕐 15.09.2026 15:25:57
![12_disque_e_monte](https://hackmd.io/_uploads/Sk7OPkuFGg.png)
 le volume **Sauvegarde (E:)** est monté et prêt ✅

### 2.2 Copie des données

La copie a été faite avec **robocopy** plutôt qu'un simple glisser-déposer : l'outil gère les arborescences profondes, reprend sur erreur, et surtout produit un **journal de copie** qui constitue une preuve exploitable.

```
robocopy "C:\Users\Sophie Martin\Documents" "E:\SAUVEGARDE-PC-COMPTA-01\Documents" /E /COPY:DAT /R:1 /W:1 /LOG+:"E:\journal-sauvegarde.txt" /TEE
```

Options retenues : `/E` copie tous les sous-dossiers y compris vides, `/COPY:DAT` copie les données, attributs et horodatages, `/R:1 /W:1` limite les tentatives en cas d'erreur, `/LOG+` écrit le journal sur le support de sauvegarde et `/TEE` affiche la progression à l'écran.

**Choix technique important :** `/COPY:DAT` ne copie volontairement **pas** les permissions NTFS. Après réinstallation, le compte de Sophie Martin recevra un nouvel identifiant de sécurité (SID) ; des permissions copiées de l'ancien système désigneraient un compte inexistant et rendraient les fichiers inaccessibles. On sauvegarde les données, pas les droits.

L'inventaire est copié sur le support au passage, pour être disponible pendant la reconfiguration.

🕐 15.09.2026 15:33:06
![13_contenu_sauvegarde](https://hackmd.io/_uploads/HksRD1utMg.png)
 contenu complet de la sauvegarde sur `E:` : Documents, Bureau, Images, Téléchargements et l'inventaire ✅ **(preuve de sauvegarde avant réinstallation)**

---

## 3. Vérification de l'intégrité de la sauvegarde

Constater qu'un fichier est présent ne prouve pas qu'il est intact : il peut être tronqué ou corrompu par la copie. La vérification repose donc sur des **empreintes de hachage SHA-256**, calculées sur chaque fichier source et sur sa copie, puis comparées. Deux fichiers identiques au bit près produisent la même empreinte ; la moindre différence change complètement le résultat.

Le rapport d'empreintes est enregistré **sur le support externe**, afin de survivre au formatage du disque système et de servir de référence après la restauration.

🕐 15.09.2026 15:38:00
![14_verif_sans_privileges_incident](https://hackmd.io/_uploads/H181_1dYzg.png)
 premier passage : "Accès refusé" sur la source, toutes les lignes marquées "SOURCE ABSENTE"

> **Incident :** la comparaison échouait alors que les fichiers existaient. **Cause :** le script était lancé depuis une session PowerShell non élevée, qui n'a pas le droit de lire le profil d'un autre utilisateur ; le test d'existence échouait silencieusement et chaque fichier était déclaré absent. **Résolution :** exécution du script via un lanceur `.bat` lancé en tant qu'administrateur. C'est un piège classique : un contrôle d'intégrité qui échoue par manque de droits peut être pris à tort pour un problème de données.

🕐 15.09.2026 16:15:16
![15_verif_integrite_avant_9_sur_9](https://hackmd.io/_uploads/HkWxOkuFMg.png)
**9 fichiers vérifiés, 9 identiques** ✅ — la sauvegarde est conforme aux originaux

---

## 4. Réinstallation propre du système

### 4.1 Mise à l'abri du support de sauvegarde

Avant toute réinstallation, le support contenant la sauvegarde est **détaché** de la machine. C'est l'équivalent de débrancher le disque dur externe : cela élimine tout risque de formater le mauvais disque pendant l'installation.

🕐 15.09.2026 16:21:54
![16_vm_settings_disque_sauvegarde_present](https://hackmd.io/_uploads/H1olO1_KGg.png)
le disque de sauvegarde avant détachement

🕐 15.09.2026 16:22:15
![17_vm_settings_disque_detache](https://hackmd.io/_uploads/BkbWukdKze.png)
 configuration après retrait : il ne reste que le disque système ✅

🕐 15.09.2026 16:22:32
![18_cdrom_iso_connecte](https://hackmd.io/_uploads/B1KWuJuFfg.png)
 support d'installation Windows 10 Pro connecté au démarrage

### 4.2 Installation

🕐 16.09.2026 08:39:14
![19_boot_manager_cdrom](https://hackmd.io/_uploads/HyBMu1dtMg.png)
 démarrage forcé sur le lecteur optique via le Boot Manager

🕐 16.09.2026 08:43:10
![20_installation_langue_suisse](https://hackmd.io/_uploads/r1WQOJdKfx.png)
— langue Français, format horaire et clavier **Français (Suisse)**

🕐 16.09.2026 08:44:08
![21_edition_windows10_pro](https://hackmd.io/_uploads/HJdXu1OFGe.png)
 édition **Windows 10 Professionnel**, identique à celle du poste d'origine

🕐 16.09.2026 08:47:04
![22_partitions_avant_suppression](https://hackmd.io/_uploads/HJMEu1utzl.png)
 les quatre partitions de l'ancien système (Système, MSR, Principale, Récupération). On note aussi qu'un seul disque est listé : le support de sauvegarde est bien hors de portée

🕐 16.09.2026 08:48:45
![23_espace_non_alloue_62go](https://hackmd.io/_uploads/Sy2Nd1utMl.png)
 après suppression des quatre partitions : **62 Go d'espace non alloué** ✅ — c'est ce qui distingue une réinstallation propre d'une simple mise à niveau

🕐 16.09.2026 09:09:11
![24_oobe_compte_admin_local](https://hackmd.io/_uploads/B1EHd1uYGl.png)
 recréation du compte administrateur local `admin-local` pendant la configuration initiale

🕐 16.09.2026 09:17:51
![25_bureau_systeme_reinstalle](https://hackmd.io/_uploads/BJgUdkOtGl.png)
système fraîchement installé : bureau vierge, aucune donnée, aucun logiciel ✅

---

## 5. Réinstallation des logiciels

### 5.1 Rebranchement du support

🕐 16.09.2026 09:25:07
![26_utiliser_disque_existant](https://hackmd.io/_uploads/S1lv_JOYzx.png)
ajout du disque avec l'option **"Use an existing virtual disk"** (et non "créer un nouveau disque", qui écraserait la sauvegarde)

🕐 16.09.2026 09:26:05
![27_chemin_disque_sauvegarde](https://hackmd.io/_uploads/rkIPdJOYzx.png)
 sélection du fichier `disque-sauvegarde.vmdk` conservé pendant la réinstallation

🕐 16.09.2026 09:29:15
![28_disque_e_retrouve](https://hackmd.io/_uploads/SkW_dJdKMl.png)
 le lecteur **Sauvegarde (E:)** est de retour, intact, après un formatage complet du disque système ✅

### 5.2 Logiciels

La liste des logiciels à réinstaller vient de la section "Logiciels installés" de l'inventaire : aucun besoin de se fier à la mémoire.

| Logiciel | Usage |
|---|---|
| LibreOffice | Suite bureautique |
| Adobe Acrobat Reader | Lecture des PDF comptables |
| 7-Zip | Gestion des archives |

🕐 16.09.2026 10:37:18
![29_logiciels_reinstalles](https://hackmd.io/_uploads/By0uOJuYze.png)
 les trois logiciels réinstallés (7-Zip visible dans la barre des tâches) ✅

---

## 6. Restauration des données et vérification

### 6.1 Prérequis : recréer le compte utilisateur

La restauration suppose que le profil de destination existe. Le compte de Sophie Martin a donc été recréé à l'identique, puis une première connexion a été effectuée pour que Windows génère son profil sur le disque.

🕐 16.09.2026 10:43:59
![30_compte_sophie_recree](https://hackmd.io/_uploads/rJiY_yuKfl.png)
 compte local **Sophie Martin** recréé

### 6.2 Restauration

La restauration est l'opération inverse de la sauvegarde, avec le même outil et les mêmes options, et son propre journal.

🕐 16.09.2026 10:47:30
![31_script_restauration](https://hackmd.io/_uploads/SkV9ukuYfg.png)
 script de restauration : robocopy du support externe vers le profil reconstitué

🕐 16.09.2026 10:49:23
![32_restauration_effectuee](https://hackmd.io/_uploads/r1A9dyOtMg.png)
 documents et Bureau restaurés : `Clients`, `dossier compta.pdf`, `facture-client-septembre.txt`, `suivi-tva-2026.csv`, `courrier-client-dupont.odt`, `notes-rdv.txt` ✅

### 6.3 Vérification d'intégrité après restauration

Le même script de vérification est relancé, cette fois entre la sauvegarde et les fichiers restaurés. Le raisonnement est le suivant : la sauvegarde a été prouvée identique aux originaux à l'étape 3 ; si les fichiers restaurés sont identiques à la sauvegarde, alors ils sont identiques aux originaux.

🕐 16.09.2026 10:53:54
![33_verif_integrite_apres_9_sur_9](https://hackmd.io/_uploads/B1BoOyOtfe.png)
 **9 fichiers vérifiés, 9 identiques** ✅

Les empreintes sont rigoureusement les mêmes qu'avant la réinstallation : par exemple `courrier-client-dupont.odt` affiche `8BD33A2D93CEB0C1...` dans les deux rapports. Les données ont traversé un formatage complet du disque système sans la moindre altération.

---

## 7. Reconfiguration des paramètres essentiels

L'inventaire réalisé à l'étape 1 sert ici de référence : chaque paramètre relevé avant la migration est remis en place.

| Paramètre | Valeur d'origine | État après reconfiguration |
|---|---|---|
| Nom de machine | `PC-COMPTA-01` | Rétabli |
| Groupe de travail | `CABINET-COMPTA` | Rétabli |
| Adresse IP | 192.168.20.237 (fixe) | Rétablie |
| Masque / passerelle | /24 — 192.168.20.1 | Rétablis |
| DNS | 8.8.8.8 et 8.8.4.4 | Rétablis |
| Imprimante | Imprimante du cabinet (pilote Microsoft Print to PDF) | Recréée sous le nom `Imprimante'Cabinet'Compta` |
| Comptes | `admin-local` (administrateur), `Sophie Martin` (standard) | Recréés |

🕐 16.09.2026 11:06:52
![34_renommage_pc_compta_01](https://hackmd.io/_uploads/S1le2dkOYze.png)
 nom de machine rétabli à `PC-COMPTA-01`, redémarrage demandé

🕐 16.09.2026 11:09:38
![35_ip_dhcp_apres_reinstallation](https://hackmd.io/_uploads/H1Yndk_YGl.png)
 état réseau juste après réinstallation : le poste a reçu une adresse DHCP différente (192.168.20.88), ce qui illustre bien la perte de configuration provoquée par la réinstallation

🕐 16.09.2026 11:12:03
![36_ip_fixe_reconfiguree](https://hackmd.io/_uploads/SJkTd1dFzx.png)
 configuration IPv4 manuelle rétablie à l'identique de l'inventaire

🕐 16.09.2026 11:13:37
![37_imprimante_reconfiguree](https://hackmd.io/_uploads/HJ86_1_YGg.png)
imprimante du cabinet recréée et visible dans la liste ✅

---

## 8. Procédure de migration réutilisable

La procédure ne se limite pas à ce document : les scripts utilisés ont été rassemblés sur le **support externe**, prêts à resservir sur un autre poste.

| Script | Rôle | Quand l'exécuter |
|---|---|---|
| `inventaire.bat` | Génère l'inventaire complet (machine, réseau, comptes, imprimantes, logiciels, données) | Avant toute opération, sur le poste source |
| `sauvegarde.bat` | Copie le profil utilisateur vers le support externe avec journal robocopy | Après validation de l'inventaire |
| `verif-integrite.ps1` + `verif.bat` | Calcule et compare les empreintes SHA-256 | Après la sauvegarde, puis après la restauration |
| `restauration.bat` | Recopie les données du support vers le profil reconstitué | Après réinstallation et recréation du compte |

🕐 16.09.2026 11:00:59
![38_boite_a_outils_scripts_migration](https://hackmd.io/_uploads/S106OJ_YMx.png)
 la boîte à outils rassemblée dans `E:\Scripts-migration`, accompagnée de l'inventaire ✅

**Leçon tirée de l'expérience :** les scripts se trouvaient initialement sur le Bureau du poste source et ont été effacés par la réinstallation. Seuls leurs résultats, enregistrés sur le support externe, ont survécu. Dans une procédure de migration, les outils doivent être stockés sur le support de sauvegarde, jamais sur le disque qui va être formaté.

### Mode opératoire type (à rejouer sur un autre poste)

1. Brancher le support externe et y copier la boîte à outils
2. Exécuter `inventaire.bat` en administrateur, vérifier que la section "Données utilisateur" est bien remplie (adapter le chemin du profil si nécessaire)
3. Exécuter `sauvegarde.bat` en administrateur, conserver le journal de copie
4. Exécuter `verif.bat` en administrateur et exiger « Identiques = Total » avant d'aller plus loin
5. **Détacher physiquement le support de sauvegarde**
6. Réinstaller le système en supprimant toutes les partitions du disque système
7. Recréer le compte administrateur, réinstaller les logiciels listés dans l'inventaire
8. Recréer le compte utilisateur et ouvrir sa session une fois pour générer son profil
9. Rebrancher le support, exécuter `restauration.bat`, puis `verif.bat` pour contrôler l'intégrité
10. Reconfigurer nom de machine, groupe de travail, réseau et imprimantes d'après l'inventaire

---

## Notes et incidents rencontrés

| Incident | Cause | Résolution |
|---|---|---|
| L'inventaire ne contenait aucune donnée utilisateur | Le script visait `C:\Users\s.martin` alors que le dossier de profil s'appelle `C:\Users\Sophie Martin` (Windows le nomme d'après le nom du compte) | Vérification du nom réel avec `dir C:\Users` puis correction du chemin |
| VMware refuse de créer le fichier du disque de sauvegarde ("cannot be written to") | Dossier de la VM non inscriptible au moment de l'opération | Création du fichier `.vmdk` à un autre emplacement, VMware ne gardant qu'une référence de chemin |
| Vérification d'intégrité renvoyant "SOURCE ABSENTE" sur tous les fichiers | Script lancé sans privilèges administrateur, donc sans droit de lecture sur le profil d'un autre utilisateur | Exécution via un lanceur `.bat` en tant qu'administrateur |
| Scripts de migration perdus après la réinstallation | Ils étaient stockés sur le Bureau du disque système formaté | Boîte à outils déplacée sur le support externe, où elle survit à l'opération |

## Points de vigilance

- **Windows non activé** : environnement de test sans licence. Une licence Windows 10 Pro devra être appliquée avant toute mise en production.
- **Conflit d'adresse IP** : le poste migré est un clone du poste d'origine et porte la même adresse fixe (192.168.20.237). Les deux machines ne doivent jamais fonctionner simultanément sur le réseau, sous peine de conflit d'adresse.
- **Nom de l'imprimante** : l'imprimante a été recréée sous le nom `Imprimante'Cabinet'Compta`, légèrement différent de l'original `Imprimante-Cabinet-Compta`. À harmoniser si la cohérence des noms importe pour l'exploitation.
- **Permissions non sauvegardées** : le choix de ne pas copier les ACL est volontaire et nécessaire après réinstallation, mais il implique que les droits d'accès particuliers (partages, restrictions par dossier) doivent être reconfigurés manuellement s'il en existait.

## Conclusion

La migration du poste `PC-COMPTA-01` est complète et vérifiée. Les données de l'utilisatrice ont été inventoriées, sauvegardées sur support externe, contrôlées par empreintes SHA-256, puis restaurées après une réinstallation intégrale du système avec suppression de toutes les partitions. Le contrôle d'intégrité final montre neuf fichiers sur neuf strictement identiques à leurs originaux, empreintes à l'appui. Les logiciels et les paramètres essentiels (nom de machine, groupe de travail, adresse IP fixe, DNS, imprimante, comptes) ont été rétablis d'après l'inventaire, et la procédure est accompagnée d'une boîte à outils scriptée réutilisable sur d'autres postes.





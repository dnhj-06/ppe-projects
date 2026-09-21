---
title: Projet 1 - Fiche de mise en service

---

---
title: PPE Projet 1 — Poste PC-COMPTA-01 (cabinet comptable)

---

# Fiche de mise en service — PC-COMPTA-01

**Projet 1 — Poste de travail pour un cabinet comptable — Classe E1B**


## Contexte

| Élément | Détail |
|---|---|
| Client | Cabinet comptable, 3 personnes |
| Poste destiné à | Sophie Martin, nouvelle collaboratrice |
| Machine | `PC-COMPTA-01` (VM VMware Workstation) |
| OS | Windows 10 Professionnel (64 bits) |
| Objectif | Poste opérationnel dès le lundi matin |

---

## 1. Installation du système d'exploitation

Support d'installation : fichier ISO fourni. VM créée avec 4 Go de RAM, 2 processeurs, disque de 62 Go, carte réseau en mode **Bridged** (pont) pour que le poste apparaisse comme une machine à part entière sur le réseau local.

🕐 14.09.2026 09:12:23
![01_support_installation_iso](https://hackmd.io/_uploads/HJR-Z8HYMx.png)
 ISO Windows présent sur le bureau hôte, support d'installation prêt

🕐 14.09.2026 09:14:04
![02_creation_vm](https://hackmd.io/_uploads/B15GZIBFfl.png)
 création de la nouvelle VM dans VMware Workstation

🕐 14.09.2026 09:14:46
![03_assistant_vm_iso_detecte](https://hackmd.io/_uploads/SkEB-IBtGe.png)
assistant VM, ISO monté : **"Windows 10 x64 detected"**

🕐 14.09.2026 09:15:28
![04_assistant_vm_capacite_disque](https://hackmd.io/_uploads/H1wLbUBtMg.png)
capacité disque fixée à **62 Go**

🕐 14.09.2026 09:17:17
![05_vm_reseau_bridge](https://hackmd.io/_uploads/r1-vZ8HKGe.png)
 carte réseau configurée en mode **Bridged** (connexion directe au réseau physique)

🕐 14.09.2026 09:16:30
![06_installation_langue_fr](https://hackmd.io/_uploads/SkAvbUHKfe.png)
 installation Windows, langue **Français (France) sur le screen mais Français suisse en réalité**

🕐 14.09.2026 09:25:33
![07_installation_edition_win10_pro](https://hackmd.io/_uploads/ryhd-IBKzl.png)
 sélection de l'édition **Windows 10 Professionnel**

🕐 14.09.2026 09:26:11
![08_installation_partition_disque](https://hackmd.io/_uploads/B1XyzUSFGg.png)
 installation sur le disque de 62 Go non alloué

🕐 14.09.2026 09:36:45
![09_oobe_region_suisse](https://hackmd.io/_uploads/ryAkMIrFMg.png)
configuration initiale (OOBE), région **Suisse**

🕐 14.09.2026 09:45:03
![10_oobe_choix_configuration](https://hackmd.io/_uploads/SyEmG8HYzx.png)

 écran "Comment souhaitez-vous configurer ?"

🕐 14.09.2026 09:45:35
![11_oobe_config_organisation](https://hackmd.io/_uploads/SkvHGUHtGl.png)
 choix **"Configurer pour une organisation"** (plutôt qu'un compte Microsoft personnel)

🕐 14.09.2026 09:46:41
![12_oobe_connexion_microsoft](https://hackmd.io/_uploads/S1zwzUBKGx.png)
 écran de connexion Microsoft affiché — **non utilisé**, contournement via le lien "Rejoindre le domaine à la place" pour créer un compte local

🕐 14.09.2026 09:47:23
![13_oobe_nom_admin_local](https://hackmd.io/_uploads/BypDf8HKGe.png)
 création du compte local **`admin-local`**

🕐 14.09.2026 09:47:42
![14_oobe_creation_mot_de_passe](https://hackmd.io/_uploads/BJTdGUSKzg.png)
 définition du mot de passe du compte

🕐 14.09.2026 09:52:46
![15_bureau_installation_terminee](https://hackmd.io/_uploads/HJRKGLHYfl.png)
bureau Windows 10 Pro opérationnel ✅

> ⚠️ **Point de vigilance :** Windows n'est pas activé (environnement de test/VM sans licence). À traiter avant toute mise en production réelle — n'empêche pas la validation du reste de la mise en service.

---

## 2. Comptes utilisateurs

| Compte | Type | Rôle |
|---|---|---|
| `admin-local` | Compte local | Administrateur |
| `s.martin` (Sophie Martin) | Compte local | Utilisateur standard |

**Politique de mot de passe appliquée :** 8 caractères minimum, présence d'une majuscule et d'un chiffre. Deux mots de passe distincts et conformes ont été définis pour les deux comptes (valeurs réelles volontairement omises de ce document, conformément aux bonnes pratiques de sécurité).

🕐 14.09.2026 09:56:30
![16_compte_admin_local_confirme](https://hackmd.io/_uploads/rkWiG8rFze.png)
 **Administrateur** (compte local)

🕐 14.09.2026 09:59:34
📸 ![17_ecran_famille_autres_utilisateurs_avant_ajout](https://hackmd.io/_uploads/Sk3sMIBtMe.png)
 écran "Famille et autres utilisateurs" avant ajout du second compte

🕐 14.09.2026 10:29:27
![18_compte_sophie_martin_ajoute](https://hackmd.io/_uploads/SyvhGIStfx.png)
 compte **Sophie Martin** ajouté en utilisateur standard ✅

🕐 14.09.2026 10:44:21
![19_ecran_connexion_deux_comptes](https://hackmd.io/_uploads/By0TGLHFze.png)
écran de connexion listant les deux comptes (`admin-local` et Sophie Martin)

---

## 3. Configuration réseau

| Paramètre | Valeur |
|---|---|
| Nom de la machine | `PC-COMPTA-01` |
| Groupe de travail | `CABINET-COMPTA` |
| Attribution IP | Automatique (DHCP) |
| Adresse IPv4 obtenue | 192.168.20.237 |
| Interface | Intel 82574L Gigabit (Bridged) |

Le poste a été laissé en DHCP plutôt qu'en IP fixe : cohérent avec un petit cabinet de 3 postes sans serveur dédié. Un passage en IP fixe ne serait justifié que si une ressource partagée (serveur de fichiers, serveur d'impression) l'exigeait.

🕐 14.09.2026 10:41:27
![20_nom_appareil_avant_renommage](https://hackmd.io/_uploads/SyblQLBtze.png)
 nom d'appareil par défaut avant renommage : **DESKTOP-OCNR23S**

🕐 14.09.2026 10:43:01
![21_renommage_pc_compta_01_groupe_travail](https://hackmd.io/_uploads/rJyZXLBFzl.png)
 renommage en **PC-COMPTA-01**, groupe de travail **CABINET-COMPTA**

🕐 14.09.2026 10:43:50
![22_redemarrage_en_cours](https://hackmd.io/_uploads/B1-zmIBFzx.png)
redémarrage requis pour appliquer le changement de nom

🕐 14.09.2026 10:45:29
![23_nom_appareil_confirme_pc_compta_01](https://hackmd.io/_uploads/rksXmIrFGl.png)
 nom confirmé après redémarrage : **PC-COMPTA-01** ✅

🕐 14.09.2026 10:46:43
![24_configuration_ip_dhcp](https://hackmd.io/_uploads/S1P47UrKMg.png)
 adresse IPv4 obtenue automatiquement : **192.168.20.237**

Pour répondre explicitement à la consigne "configurer l'IP" (et pas seulement constater l'attribution automatique), l'adresse a ensuite été reprise et **figée en configuration manuelle** avec les mêmes valeurs que le bail DHCP en cours.

🕐 14.09.2026, peu après 12:11 (finalisation de l'étape 3)
![37_dialogue_ip_manuel_vide](https://hackmd.io/_uploads/BkqsYUStGl.png)

ouverture de "Modifier les paramètres IP", passage en **Manuel**, champs vides avant saisie

🕐 14.09.2026, peu après 12:11
![38_verification_passerelle_avant_config](https://hackmd.io/_uploads/BJACKIBKfx.png)

vérification préalable via `ipconfig` : passerelle par défaut confirmée **192.168.20.1**, masque **255.255.255.0**

🕐 14.09.2026, peu après 12:11
![39_saisie_ip_manuelle_valeurs](https://hackmd.io/_uploads/rJ61qLBFGx.png)

saisie des valeurs manuelles : IP 192.168.20.237, préfixe /24, passerelle 192.168.20.1, DNS préféré 8.8.8.8, autre DNS 8.8.4.4

🕐 14.09.2026, peu après 12:11
![40_verification_ip_ping_apres_config](https://hackmd.io/_uploads/HkOx9UHYGe.png)

vérification après enregistrement : `ipconfig` confirme la config manuelle, `ping google.com` réussit (3/3 réponses reçues) ✅

---

## 4. Logiciels installés

| Logiciel | Usage |
|---|---|
| LibreOffice | Suite bureautique (traitement de texte, tableur) |
| Adobe Acrobat Reader | Lecture de PDF (formulaires comptables, factures) |
| 7-Zip | Gestion des archives (réception de dossiers de pièces comptables) |

Installation manuelle via navigateur, depuis les sites officiels des éditeurs (voir incident winget ci-dessous).

🕐 14.09.2026 11:34:36
![25_7zip_deja_installe](https://hackmd.io/_uploads/SyTrX8SYzl.png)
 7-Zip installé, visible dans la recherche Windows

🕐 14.09.2026 11:35:45
![26_bureau_logiciels_installes](https://hackmd.io/_uploads/BkYLXLBKzl.png)
 bureau avec LibreOffice et Adobe Acrobat installés (Adobe Express Photos déjà désinstallé) ✅

---

## 5. Imprimante réseau partagée

Imprimante représentant l'imprimante réseau du cabinet ajoutée manuellement (pilote Microsoft Print to PDF, port fichier), sous le nom **`Imprimante-Cabinet-Compta`**.

🕐 14.09.2026 11:53:03
![27_imprimantes_scanners_avant_ajout](https://hackmd.io/_uploads/H10DQLrtMl.png)
 écran "Imprimantes et scanners" avant ajout

🕐 14.09.2026 11:54:12
![28_ajout_imprimante_options_avancees](https://hackmd.io/_uploads/Hy1YX8BFMe.png)
ajout d'une imprimante, options avancées ("imprimante locale ou réseau avec paramètres manuels")

🕐 14.09.2026 11:54:40
![29_choix_port_file](https://hackmd.io/_uploads/SJnt7ISKMl.png)
 port choisi : **FILE: (Impression dans un fichier)**

🕐 14.09.2026 11:55:24
![30_nom_imprimante_cabinet_compta](https://hackmd.io/_uploads/S1t97LStMg.png)
 nom de l'imprimante : **Imprimante-Cabinet-Compta**

🕐 14.09.2026 11:55:50
![31_liste_imprimantes_ajout_confirme](https://hackmd.io/_uploads/HkNimIBKfx.png)
 imprimante ajoutée, visible dans la liste ✅

### Test d'impression

🕐 14.09.2026 12:03:39
![32_libreoffice_texte_test](https://hackmd.io/_uploads/HkJ3mIHtMl.png)
 document de test créé dans LibreOffice Writer ("Teste impression")

🕐 14.09.2026 12:04:07
![33_fenetre_impression_imprimante_selectionnee](https://hackmd.io/_uploads/Sy037USYMx.png)
 fenêtre d'impression, **Imprimante-Cabinet-Compta** sélectionnée

> **Incident :** le premier fichier généré (`teste-impression.pdf`) faisait **0 octet** — fichier corrompu. **Cause :** la fenêtre d'enregistrement a été fermée avant la fin du traitement du job d'impression. **Résolution :** nouvel essai en laissant le temps au job de se terminer avant de rouvrir le fichier.

🕐 14.09.2026 12:08:27
![34_fichier_pdf_genere_preuve_succes](https://hackmd.io/_uploads/Hysa7IBYMe.png)
second essai : fichier PDF valide généré sur le bureau ✅

---

## 6. Sécurité

| Point vérifié | État |
|---|---|
| Mises à jour Windows | À jour, aucune mise à jour en attente ✅ |
| Antivirus (Windows Defender) | Actif, aucune action requise ✅ |
| Pare-feu | Actif, aucune action requise ✅ |
| Protection du compte | Aucune action requise ✅ |
| Contrôle des applications et du navigateur | Aucune action requise ✅ |

🕐 14.09.2026 12:11:04
![35_windows_update_a_jour](https://hackmd.io/_uploads/ryU0XLStfg.png)
 Windows Update : **"Aucune mise à jour disponible"**

🕐 14.09.2026 12:11:17
![36_zones_protection_tout_vert](https://hackmd.io/_uploads/H17kE8rKGl.png)
 Zones de protection Windows Security, toutes au vert ✅

---

## Notes / Incidents rencontrés (rapport de diagnostic)

| Incident | Cause | Résolution |
|---|---|---|
| Téléchargement de LibreOffice redirigé vers une fiche payante du Microsoft Store | Recherche effectuée depuis la barre de recherche Windows plutôt que la barre d'adresse du navigateur | Saisie directe de l'URL officielle `libreoffice.org` dans Edge — LibreOffice est gratuit, aucun paiement effectué |
| `winget --version` renvoie `CommandNotFoundException` dans PowerShell | winget (Windows Package Manager) n'est pas préinstallé par défaut sur Windows 10 Pro (contrairement à Windows 11) | Installation manuelle des 3 logiciels via téléchargement navigateur, plus simple ici que d'installer "App Installer" via le Store sans compte Microsoft |
| Premier test d'impression : fichier PDF généré à 0 octet | Fenêtre d'enregistrement du job d'impression fermée avant la fin du traitement | Nouvel essai en laissant le job d'impression se terminer complètement avant d'ouvrir le fichier |
| Logiciel "Adobe Express Photos" installé sans consentement explicite | Bundlé silencieusement par l'installateur Adobe Acrobat Reader | Identifié puis désinstallé (`Paramètres > Applications`) pour garder un poste propre |

## Points de vigilance avant remise du poste

- **Windows non activé** : environnement de test sans licence. Une licence Windows 10 Pro devra être appliquée avant mise en production réelle.
- **Offres tierces à l'installation** : penser à décocher les logiciels additionnels proposés par les installateurs (Adobe notamment) pour éviter tout bundle indésirable à l'avenir.

## Conclusion

Le poste `PC-COMPTA-01` est opérationnel : OS installé et configuré, comptes administrateur et utilisateur standard créés avec politique de mot de passe appliquée, poste intégré au réseau local du cabinet (DHCP), logiciels bureautiques de base installés, imprimante réseau ajoutée et testée avec succès, mesures de sécurité de base vérifiées et actives. Prêt à être remis à Sophie Martin dès le lundi matin, sous réserve de l'activation de la licence Windows.

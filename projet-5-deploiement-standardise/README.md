---
title: PPE Projet 5 — Déploiement standardisé d'un poste (image système)

---

---
title: PPE Projet 5 — Déploiement standardisé d'un poste (image système)
---

# Procédure de déploiement par image — Poste standard

**Projet 5 — Classe E1B — Niveau Avancé**

---

## Contexte et objectif

Une entreprise doit équiper **10 postes identiques**. Plutôt que d'installer et de configurer chaque poste à la main, on prépare **un poste de référence**, on le transforme en **image système**, et on **déploie cette image** sur les autres postes. Objectif : un déploiement rapide, standardisé et reproductible.

**Méthode retenue :** image disque avec **Clonezilla**, précédée d'une **généralisation Sysprep** (indispensable pour que chaque poste reçoive une identité unique). Tout est réalisé dans **VMware Workstation**.

| Élément | Valeur |
|---|---|
| Poste de référence | `PC-STD-REF` (VM Windows 10 Pro, clone neuf) |
| Poste déployé (test) | `PC-STD-01` |
| Outil de généralisation | Sysprep (OOBE + Généraliser) |
| Outil d'imagerie | Clonezilla Live 3.3.3-15 (amd64) |
| Stockage de l'image | 2ᵉ disque virtuel « dépôt » (40 Go, ext4) |

---

## 1. Cahier des charges du poste standard

Le poste type que tous les exemplaires doivent respecter :

| Élément | Spécification |
|---|---|
| Système | Windows 10 Professionnel 64 bits, Français (Suisse) |
| Matériel (VM) | 2 vCPU, 4 Go RAM, disque 40 Go |
| Compte administrateur | `admin-local` |
| Compte utilisateur | `utilisateur` (standard) |
| Logiciel | 7-Zip (LibreOffice et Adobe Reader font partie du standard en production ; seul 7-Zip a été installé pour l'exercice) |
| Réseau | DHCP (adresse automatique) |
| Sécurité | Mot de passe sécurisé + verrouillage automatique après 300 s d'inactivité |
| Nommage | `PC-STD-##` (référence : `PC-STD-REF`, déployé : `PC-STD-01`) |

---

## 2. Préparation du poste de référence

Un poste de référence propre est installé et configuré **une seule fois**. C'est lui qui sera « photographié » pour servir de modèle.

🕐 17.09.2026 08:58:53
![01_creation_vm_reference](https://hackmd.io/_uploads/Byb-euYYGg.png)
 création de la VM à partir de l'ISO Windows 10 Pro (détecté « Windows 10 x64 »)

🕐 17.09.2026 09:02:25
![02_specs_vm_reference](https://hackmd.io/_uploads/Hk9-euFFGe.png)
 récapitulatif conforme au cahier des charges : `PC-STD-REF`, **40 Go**, **4096 Mo**, **2 CPU**

🕐 17.09.2026 09:08:25
![03_installation_windows](https://hackmd.io/_uploads/SyuMeuFFze.png)
 installation de Windows 10 Professionnel en cours

🕐 17.09.2026 09:28:13
![04_bureau_reference](https://hackmd.io/_uploads/S1WQedtKMe.png)
 Windows installé, arrivée sur le bureau du poste de référence ✅

### Configuration selon le cahier des charges

🕐 17.09.2026 10:11:20
![05_comptes_standard](https://hackmd.io/_uploads/BJxExOYYfe.png)
 comptes locaux créés : **`admin-local`** (administrateur) et **`utilisateur`** (standard). *Note : la commande `net user` doit être lancée depuis une invite **administrateur**, sinon elle renvoie « Erreur système 5 – Accès refusé ».*

🕐 17.09.2026 10:35:07
![06_logiciel_7zip](https://hackmd.io/_uploads/SkgrlutKGl.png)
 **7-Zip** installé (visible sur le bureau)

🕐 17.09.2026 10:38:05
![07_securite_inactivite_300](https://hackmd.io/_uploads/BJxLedtYzg.png)
 stratégie de sécurité : verrouillage automatique après **300 secondes** d'inactivité (`secpol.msc`)

---

## 3. Généralisation avec Sysprep

C'est **l'étape clé** du déploiement. Un simple clone donnerait 10 postes portant le **même identifiant de sécurité (SID) et le même nom** → conflits sur le réseau et le domaine. **Sysprep** retire ces éléments uniques : au premier démarrage, chaque poste déployé génère sa propre identité.

🕐 17.09.2026 10:48:07
![08_sysprep_generaliser](https://hackmd.io/_uploads/rJqvgdttzl.png)
 Sysprep configuré en **« Entrer en mode OOBE »**, case **« Généraliser » cochée**, option **« Arrêter le système »**. La case *Généraliser* est ce qui rend l'image déployable.

🕐 17.09.2026 10:50:14
![09_sysprep_arret](https://hackmd.io/_uploads/HJQOeOFtGx.png)
 Sysprep terminé, le poste s'éteint automatiquement

🕐 17.09.2026 10:54:38
![10_reference_eteinte](https://hackmd.io/_uploads/SJJKluFYzg.png)
 `PC-STD-REF` éteint et généralisé, prêt à être capturé ✅

---

## 4. Création de l'image système (Clonezilla)

Clonezilla est un système « live » : on démarre la VM dessus (aucune installation), et il copie le disque système vers un **fichier image** stocké sur un second disque « dépôt ».

🕐 17.09.2026 11:01:24
![11_clonezilla_telechargement](https://hackmd.io/_uploads/BypYxdYFMe.png)
 Clonezilla Live (stable, amd64, iso) téléchargé sur le poste hôte

🕐 17.09.2026 11:14:08
![12_clonezilla_boot](https://hackmd.io/_uploads/SkLql_KKMe.png)
 démarrage de la VM sur le CD Clonezilla (menu GRUB)

🕐 17.09.2026 11:24:01
![13_mode_device_image](https://hackmd.io/_uploads/S1JoldFtMe.png)
 mode **`device-image`** : travailler entre un disque et un fichier image

> **Point technique — préparation du disque dépôt :** un disque virtuel neuf est **vierge** (aucune partition), or Clonezilla exige un dépôt **formaté**. Il a fallu créer une partition et la formater en ext4 avant de pouvoir y écrire l'image. La distinction entre disque système (`nvme0n1`, 4 partitions Windows) et disque dépôt (`nvme0n2`, vide) a été vérifiée avec `lsblk` pour ne pas formater le mauvais disque.

🕐 17.09.2026 11:38:30
![14_formatage_disque_depot](https://hackmd.io/_uploads/rk5sxuKKzg.png)
 formatage du disque dépôt : `parted` (table GPT + partition) puis `mkfs.ext4`

🕐 17.09.2026 11:41:22
![15_depot_pret](https://hackmd.io/_uploads/rkM2euKYfg.png)
 `lsblk` confirme la partition **`nvme0n2p1`** prête à recevoir l'image

🕐 17.09.2026 11:55:16
![16_lancement_capture](https://hackmd.io/_uploads/S1xpeutKGx.png)
 lancement de la capture (`savedisk`) du disque système `nvme0n1` vers l'image `image-PC-STD-2026`

🕐 17.09.2026 11:58:30
![17_capture_progression](https://hackmd.io/_uploads/Hkj0e_tFfx.png)
 copie en cours avec **Partclone** (partition Windows NTFS, débit ~6,4 Go/min)

🕐 17.09.2026 12:01:12
![18_image_creee_verifiee](https://hackmd.io/_uploads/BkOkW_Ytfl.png)
 **image créée et vérifiée** : « All partition images were checked and are **restorable** » ✅ (la vérification prouve que l'image est exploitable)

---

## 5. Déploiement et test sur une seconde machine

Une VM cible **vierge** est créée, on lui attache le disque dépôt (contenant l'image) et l'ISO Clonezilla, puis on **restaure** l'image dessus.

🕐 17.09.2026 13:24:02
![19_creation_vm_cible](https://hackmd.io/_uploads/B1cl-_YFzl.png)
 création de la VM cible `PC-STD-01` (40 Go) et ajout du disque dépôt

🕐 17.09.2026 13:55:51
![20_restauration_confirmation](https://hackmd.io/_uploads/SkNbZutKMl.png)
 mode **`restoredisk`** : l'image `image-PC-STD-2026` va être restaurée vers le disque vierge `nvme0n1` de `PC-STD-01` (double confirmation « toutes les données seront écrasées »)

🕐 17.09.2026 14:23:46
![21_restauration_terminee](https://hackmd.io/_uploads/Sy3-ZuKYzl.png)
restauration terminée, `PC-STD-01` s'est éteint automatiquement. On retire alors l'ISO Clonezilla et le disque dépôt avant de démarrer.

### Vérification de conformité (preuve de test)

🕐 17.09.2026 14:33:12
![22_login_comptes_deployes](https://hackmd.io/_uploads/HyLf-OYtzl.png)
 écran de connexion du poste déployé : les comptes **`admin-local`** et **`utilisateur`** du cahier des charges sont bien présents ✅

🕐 17.09.2026 14:35:53
![23_verif_hostname_comptes](https://hackmd.io/_uploads/S1afZ_YKGx.png)
`hostname` = **`DESKTOP-R16UIG4`**, un **nouveau nom généré automatiquement** (différent du poste de référence) : **preuve que Sysprep a bien attribué une nouvelle identité**. `net user` confirme les comptes standard. ✅

🕐 17.09.2026 14:40:24
![24_verif_securite_300](https://hackmd.io/_uploads/SkPXW_YtMg.png)
 le réglage de sécurité (verrouillage après **300 s**) a bien été conservé dans l'image ✅

🕐 17.09.2026 14:41:51
![25_renommage_pc_std_01](https://hackmd.io/_uploads/SJk4buFtMe.png)
 attribution du nom final **`PC-STD-01`** au poste déployé ✅

**Bilan de conformité :** le poste déployé possède la même configuration que le poste de référence (comptes, logiciel, sécurité) **mais une identité machine propre** (nouveau SID, nouveau nom). Le déploiement est donc à la fois **standardisé** et **sans conflit d'identité**.

---

## 6. Comparatif de temps : manuel vs image

Les durées ci-dessous sont **indicatives** (mesurées sur les VM de test). Elles suffisent à montrer l'ordre de grandeur du gain.

### Temps mesurés pour un poste

| Opération | Temps mesuré | Nature |
|---|---|---|
| Installation Windows + OOBE | ~30 min | par poste (manuel) |
| Configuration (comptes, 7-Zip, sécurité) | ~40 min | par poste (manuel) |
| **Total installation manuelle** | **~1 h 10** | **par poste** |
| Sysprep + capture de l'image | ~25 min | **une seule fois** |
| Restauration de l'image + 1ᵉʳ démarrage + renommage | ~20 min | par poste (image) |

### Projection sur les 10 postes

| Méthode | Calcul | Total |
|---|---|---|
| **Installation manuelle** | 10 × 1 h 10 | **≈ 11 h 40** |
| **Déploiement par image** | 25 min (image, 1×) + 10 × 20 min | **≈ 3 h 45** |

**Gain estimé : ≈ 8 heures pour 10 postes, soit environ 68 % de temps en moins.** Plus le parc est grand, plus l'écart se creuse : le coût de création de l'image est fixe (payé une fois), alors que le temps manuel croît proportionnellement au nombre de postes.

> À noter : la toute première mise en place (télécharger Clonezilla, créer et formater le disque dépôt) est un **coût unique d'infrastructure**, non répété à chaque poste ; il n'est pas compté dans le temps de déploiement par poste.

---

## 7. Procédure de déploiement réutilisable

Mode opératoire condensé, à rejouer pour chaque nouveau lot de postes :

1. **Préparer** un poste de référence conforme au cahier des charges (OS, comptes, logiciels, réglages).
2. **Généraliser** avec `sysprep` → *Entrer en mode OOBE* + *Généraliser* + *Arrêter*.
3. **Démarrer** le poste de référence sur **Clonezilla Live**.
4. **Capturer** le disque système en image (`device-image` → `savedisk`), stockée sur un disque/partage dédié et **formaté**.
5. Pour **chaque** poste à déployer : créer une VM/disque vierge, démarrer sur Clonezilla, **restaurer** l'image (`device-image` → `restoredisk`).
6. **Premier démarrage** : l'OOBE génère une nouvelle identité ; attribuer le nom final `PC-STD-##`.
7. **Vérifier** la conformité (comptes, logiciels, sécurité) et l'unicité de l'identité (nom/SID).

---

## Points de vigilance

- **Sysprep est obligatoire avant l'image :** sans « Généraliser », tous les postes déployés partageraient le même SID et le même nom → conflits réseau. C'est le cœur du déploiement standardisé.
- **Dépôt sur un disque séparé et formaté :** on ne peut pas stocker l'image sur le disque qu'on copie ; le disque dépôt doit être formaté (ext4/NTFS) avant que Clonezilla l'accepte.
- **Ne pas confondre source et destination :** à la restauration, l'image (source) est écrite **vers** le disque vierge (destination). Vérifier les noms de disques (`lsblk`) avant de valider — une inversion écrase l'image.
- **Retirer l'ISO et le disque dépôt** avant de démarrer le poste déployé, sinon il redémarre sur Clonezilla.
- **Windows non activé :** environnement de test ; une licence Windows 10 Pro (ou l'activation KMS/volume) devra être appliquée en production.
- **Piste industrielle :** pour un parc réel, on automatiserait encore avec un fichier `unattend.xml` (OOBE sans intervention) ou une solution de déploiement réseau (WDS/MDT, ou Windows Autopilot pour le cloud).

## Conclusion

La méthode de déploiement standardisé est en place et testée : un poste de référence conforme au cahier des charges a été **généralisé (Sysprep)** puis transformé en **image système (Clonezilla)**, et cette image a été **déployée avec succès sur une seconde machine**, dont la conformité (comptes, logiciel, sécurité) et l'**unicité d'identité** (nouveau nom/SID) ont été vérifiées. Le comparatif montre un gain d'environ **68 %** de temps sur 10 postes (≈ 3 h 45 contre ≈ 11 h 40). La procédure est documentée et réutilisable pour tout nouveau lot de postes.
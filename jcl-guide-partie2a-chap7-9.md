# JCL - Job Control Language : Guide Complet Mondial
## Partie 2A : DISP, Organisation, GDG (Chapitres 7-9)

**🔗 Repository GitHub :** [Learning Schooling Foundation - JCL](https://github.com/learning-schooling-foundation)

---

## Table des Matières - Partie 2A

7. [DISP - Gérer le Cycle de Vie en Profondeur](#7-disp---gérer-le-cycle-de-vie-en-profondeur)
8. [Organisation des Datasets](#8-organisation-des-datasets)
9. [GDG - Generation Data Groups](#9-gdg---generation-data-groups)

---

## 7. DISP - Gérer le Cycle de Vie en Profondeur

### Pourquoi DISP est CRITIQUE

**DISP (Disposition) contrôle TOUT le cycle de vie d'un dataset.**

**Un mauvais DISP peut :**
- ❌ Supprimer des données de production
- ❌ Créer des conflits d'accès (corruption de données)
- ❌ Perdre des fichiers (non catalogués)
- ❌ Faire planter un job sans raison apparente

**Dans 30 ans de carrière mainframe, 60% des erreurs JCL viennent d'un mauvais DISP.**

**C'est LE paramètre à maîtriser ABSOLUMENT.**

### Structure Complète de DISP

```jcl
DISP=(status,normal-termination,abnormal-termination)
```

**3 composantes :**

1. **Status** - État AVANT l'exécution du step
2. **Normal-termination** - Que faire si le step RÉUSSIT
3. **Abnormal-termination** - Que faire si le step PLANTE

### Status (Premier Paramètre)

**Décrit l'état du dataset AVANT que le step commence.**

| Status | Signification | Dataset existe ? | Accès |
|--------|---------------|------------------|-------|
| **NEW** | Nouveau, à créer | NON | Exclusif |
| **OLD** | Existant, accès exclusif | OUI | Exclusif |
| **SHR** | Existant, accès partagé | OUI | Partagé |
| **MOD** | Modifier (ajoute à la fin) | OUI ou NON | Exclusif |

### NEW - Créer un Nouveau Dataset

**Utilise NEW quand le dataset N'EXISTE PAS et tu veux le créer.**

```jcl
//OUTPUT DD DSN=NOUVEAU.FICHIER,
//          DISP=(NEW,CATLG),
//          UNIT=SYSDA,
//          SPACE=(TRK,(10,5))
```

**Ce qui se passe :**
1. Le système vérifie que `NOUVEAU.FICHIER` n'existe PAS
2. Si existe déjà → **JCL plante** avec "DUPLICATE DATASET NAME"
3. Si n'existe pas → Crée le dataset

**⚠️ ERREUR CLASSIQUE #1 : Utiliser NEW sur un fichier existant**

```jcl
//OUTPUT DD DSN=PROD.MASTER.DATA,DISP=(NEW,CATLG)
```

Si `PROD.MASTER.DATA` existe déjà :
```
IEF257I jobname stepname OUTPUT - DUPLICATE DATA SET NAME
JCL ERROR
```

**💡 SOLUTION :**

Soit tu supprimes d'abord le fichier :

```jcl
//STEP1 EXEC PGM=IEFBR14
//DEL   DD DSN=PROD.MASTER.DATA,DISP=(MOD,DELETE)

//STEP2 EXEC PGM=...
//OUT   DD DSN=PROD.MASTER.DATA,DISP=(NEW,CATLG)
```

Soit tu utilises `MOD` à la place de `NEW`.

### OLD - Accès Exclusif à un Dataset Existant

**Utilise OLD quand :**
- Le dataset EXISTE
- Tu veux le MODIFIER (écrire dedans)
- Tu veux un accès EXCLUSIF (personne d'autre ne peut l'utiliser)

```jcl
//UPDATE DD DSN=MASTER.FILE,DISP=OLD
```

**Ce qui se passe :**
1. Le système vérifie que `MASTER.FILE` existe
2. Si n'existe pas → **JCL plante** avec "DATASET NOT FOUND"
3. Si existe → Réserve l'accès exclusif (personne d'autre ne peut l'ouvrir)

**Pourquoi "OLD" ?**

Le nom est trompeur. Ça signifie "existant" + "accès exclusif", pas "vieux".

**Quand utiliser OLD :**

```jcl
//* Modifier un fichier
//UPDATE DD DSN=MASTER.FILE,DISP=OLD

//* Supprimer un fichier
//DEL DD DSN=OLD.FILE,DISP=(OLD,DELETE)

//* Écrire dans un fichier (écrase le contenu)
//OUTPUT DD DSN=REPORT.FILE,DISP=OLD
```

**⚠️ DANGER : OLD verrouille le fichier**

Si un autre job essaie d'accéder au même fichier avec OLD ou SHR pendant que ton job tourne :
- Il est mis en ATTENTE
- Si ton job est long → Blocage de la production !

**💡 N'utilise OLD que si tu MODIFIES vraiment le fichier.**

### SHR - Accès Partagé (Lecture Seule)

**Utilise SHR quand :**
- Le dataset EXISTE
- Tu veux seulement le LIRE
- D'autres jobs peuvent aussi le lire en même temps

```jcl
//INPUT DD DSN=REFERENCE.DATA,DISP=SHR
```

**Ce qui se passe :**
1. Le système vérifie que `REFERENCE.DATA` existe
2. Si n'existe pas → **JCL plante** avec "DATASET NOT FOUND"
3. Si existe → Ouvre en lecture seule
4. D'autres jobs peuvent aussi l'ouvrir avec SHR

**MULTIPLE jobs peuvent utiliser SHR sur le MÊME fichier simultanément.**

**Quand utiliser SHR :**

```jcl
//* Lire un fichier de référence
//INPUT DD DSN=PROD.REFERENCE.TABLE,DISP=SHR

//* Copier un fichier (source)
//SYSUT1 DD DSN=SOURCE.DATA,DISP=SHR

//* Lire un fichier master (pas de modif)
//MASTER DD DSN=CUSTOMER.MASTER,DISP=SHR
```

**⚠️ ERREUR CRITIQUE : SHR sur un fichier à modifier**

```jcl
//OUTPUT DD DSN=MASTER.FILE,DISP=SHR
```

Si ton programme ÉCRIT dans ce fichier ET un autre job le lit en SHR :
- **CORRUPTION DE DONNÉES**
- Résultats imprévisibles
- Possible crash du système

**💡 RÈGLE D'OR :**

**Lecture → SHR**  
**Écriture/Modification → OLD**

### MOD - Modifier (Ajouter à la Fin)

**MOD est SPÉCIAL : Il fonctionne que le dataset existe ou pas.**

**Si le dataset EXISTE :**
- Ouvre le fichier
- Positionne à la FIN
- Les écritures s'AJOUTENT à la fin

**Si le dataset N'EXISTE PAS :**
- Crée le dataset (comme NEW)
- Écrit dedans

```jcl
//LOG DD DSN=APPLICATION.LOG,
//       DISP=(MOD,KEEP),
//       UNIT=SYSDA,
//       SPACE=(TRK,(10,5))
```

**Cas d'usage typique : Fichiers de log**

```jcl
//STEP1 EXEC PGM=PROG1
//LOG   DD DSN=DAILY.LOG,DISP=MOD

//STEP2 EXEC PGM=PROG2
//LOG   DD DSN=DAILY.LOG,DISP=MOD

//STEP3 EXEC PGM=PROG3
//LOG   DD DSN=DAILY.LOG,DISP=MOD
```

Chaque step AJOUTE ses messages au même log.

**⚠️ ATTENTION avec MOD :**

MOD ne REMPLACE PAS le contenu. Il AJOUTE.

```jcl
//OUTPUT DD DSN=REPORT.FILE,DISP=MOD
```

Si `REPORT.FILE` contient déjà 1000 lignes, les nouvelles lignes seront ajoutées après la ligne 1000.

**💡 Pour REMPLACER le contenu :**

Soit utilise `DISP=OLD`, soit supprime d'abord puis utilise `DISP=(NEW,CATLG)`.

### Normal-Termination (Deuxième Paramètre)

**Que faire avec le dataset si le step SE TERMINE NORMALEMENT (RC=0) ?**

| Valeur | Action | Usage |
|--------|--------|-------|
| **CATLG** | Catalogue le dataset | Fichiers à garder |
| **DELETE** | Supprime le dataset | Fichiers temporaires |
| **KEEP** | Garde mais ne catalogue pas | Rare |
| **PASS** | Passe au step suivant | Temporaires intra-job |
| **UNCATLG** | Décatalogue (garde les données) | Très rare |

### CATLG - Cataloguer

**CATLG enregistre le dataset dans le catalogue système.**

```jcl
//OUTPUT DD DSN=PROD.MASTER.FILE,
//          DISP=(NEW,CATLG)
```

**Après le step :**
- Le dataset existe sur disque
- Il est catalogué → Tu peux y accéder par son nom dans d'autres jobs

**💡 UTILISE TOUJOURS CATLG pour les datasets permanents.**

**Sinon, tu ne pourras plus retrouver le fichier !**

### DELETE - Supprimer

**DELETE supprime complètement le dataset (données + catalogue).**

```jcl
//TEMP DD DSN=TEMPORARY.FILE,
//        DISP=(NEW,DELETE)
```

**Après le step (si succès) :**
- Le dataset est SUPPRIMÉ
- Espace disque libéré

**Pourquoi créer un fichier pour le supprimer ?**

**Si le step PLANTE, tu ne veux PAS supprimer le fichier (pour débugger).**

```jcl
//TEMP DD DSN=WORK.FILE,
//        DISP=(NEW,DELETE,KEEP)
              ↑      ↑       ↑
              Crée   Si OK   Si erreur
                     supprime garde
```

### PASS - Passer au Step Suivant

**PASS garde le dataset pour les steps suivants du MÊME job.**

**Utilisé avec les datasets temporaires (`&&NAME`).**

```jcl
//STEP1 EXEC PGM=PROG1
//TEMP  DD DSN=&&TEMPFILE,
//         DISP=(NEW,PASS),
//         SPACE=(TRK,(10,5))

//STEP2 EXEC PGM=PROG2
//INPUT DD DSN=&&TEMPFILE,
//         DISP=(OLD,DELETE)
```

**STEP1** crée `&&TEMPFILE` et le PASSE à **STEP2**.  
**STEP2** l'utilise puis le SUPPRIME.

**Le dataset est automatiquement supprimé à la fin du job.**

**💡 PASS est OPTIMAL pour les fichiers intermédiaires dans un job multi-steps.**

### Abnormal-Termination (Troisième Paramètre)

**Que faire si le step PLANTE (ABEND) ?**

**Si tu ne spécifies pas, il prend la même valeur que normal-termination.**

```jcl
DISP=(NEW,CATLG)
     ↑     ↑
     NEW   CATLG si succès OU échec
```

**Mais souvent, tu veux un comportement DIFFÉRENT :**

```jcl
DISP=(NEW,CATLG,DELETE)
     ↑     ↑      ↑
     NEW   Si OK  Si erreur
           garde  supprime
```

**Cas d'usage typique :**

```jcl
//OUTPUT DD DSN=REPORT.FILE,
//          DISP=(NEW,CATLG,DELETE),
//          UNIT=SYSDA,
//          SPACE=(TRK,(10,5))
```

**Si le step réussit :** Le rapport est catalogué ✅  
**Si le step plante :** Le rapport incomplet est supprimé ❌ (pas de données partielles)

### Formes Abrégées de DISP

**Forme complète :**
```jcl
DISP=(status,normal,abnormal)
```

**Forme courte (normal = abnormal) :**
```jcl
DISP=(status,disposition)
```

**Exemples :**

```jcl
DISP=(NEW,CATLG)
    = DISP=(NEW,CATLG,CATLG)

DISP=(OLD,DELETE)
    = DISP=(OLD,DELETE,DELETE)
```

**Forme ultra-courte (que status) :**

```jcl
DISP=SHR
    = DISP=(SHR,KEEP,KEEP)

DISP=OLD
    = DISP=(OLD,KEEP,KEEP)

DISP=MOD
    = DISP=(MOD,KEEP,KEEP)
```

**⚠️ ATTENTION à la forme ultra-courte :**

```jcl
DISP=NEW
    = DISP=(NEW,KEEP,KEEP)
```

Le dataset est créé mais **PAS CATALOGUÉ** → Tu ne peux plus le retrouver !

**💡 TOUJOURS spécifier au moins (NEW,CATLG).**

### Matrice de Décision DISP

**Voici comment choisir ton DISP :**

| Scénario | DISP |
|----------|------|
| Créer un fichier permanent | `(NEW,CATLG,DELETE)` |
| Lire un fichier existant | `SHR` |
| Modifier un fichier existant | `OLD` |
| Supprimer un fichier | `(OLD,DELETE)` |
| Fichier temporaire (1 step) | `(NEW,DELETE)` |
| Fichier temporaire (multi-steps) | `(NEW,PASS)` puis `(OLD,DELETE)` |
| Ajouter à un log | `MOD` |
| Remplacer un fichier | `OLD` (écrase) ou DELETE puis `(NEW,CATLG)` |

### Exemples Réels de Production

**1. Job de rapport quotidien :**

```jcl
//REPORT  JOB (ACCT),'DAILY REPORT',CLASS=A,MSGCLASS=X
//*
//* Extract data to temporary file
//*
//EXTRACT EXEC PGM=EXTRACT01
//INPUT   DD DSN=PROD.MASTER.DATA,DISP=SHR
//OUTPUT  DD DSN=&&EXTRACT,
//           DISP=(NEW,PASS),
//           SPACE=(CYL,(5,1))
//*
//* Sort the extracted data
//*
//SORT    EXEC PGM=SORT
//SORTIN  DD DSN=&&EXTRACT,DISP=(OLD,PASS)
//SORTOUT DD DSN=&&SORTED,
//           DISP=(NEW,PASS),
//           SPACE=(CYL,(5,1))
//SYSIN   DD *
  SORT FIELDS=(1,10,CH,A)
/*
//*
//* Generate report
//*
//REPORT  EXEC PGM=REPORT01
//INPUT   DD DSN=&&SORTED,DISP=(OLD,DELETE)
//OUTPUT  DD DSN=REPORT.DAILY.&SYSDATE,
//           DISP=(NEW,CATLG,DELETE),
//           UNIT=SYSDA,
//           SPACE=(TRK,(50,10),RLSE),
//           DCB=(RECFM=FB,LRECL=132,BLKSIZE=27984)
```

**Analyse :**
- `&&EXTRACT` et `&&SORTED` : Temporaires avec PASS
- `PROD.MASTER.DATA` : SHR (lecture seule)
- `REPORT.DAILY.xxx` : Catalogué si OK, supprimé si erreur

---

**2. Job de mise à jour master file :**

```jcl
//UPDATE  JOB (ACCT),'UPDATE MASTER',CLASS=A,MSGCLASS=X
//*
//* Read transactions
//*
//STEP1   EXEC PGM=UPDATE01
//TRANS   DD DSN=DAILY.TRANSACTIONS,DISP=SHR
//MASTER  DD DSN=PROD.MASTER.FILE,
//           DISP=OLD
//ERRORS  DD DSN=ERROR.LOG.&SYSDATE,
//           DISP=(MOD,CATLG),
//           UNIT=SYSDA,
//           SPACE=(TRK,(10,5),RLSE)
```

**Analyse :**
- Transactions en SHR (lecture)
- Master en OLD (modification exclusive)
- Error log en MOD (ajoute chaque jour)

### Exercice Pratique : DISP

**Trouve les erreurs dans ce job :**

```jcl
//TESTJOB JOB (ACCT),'TEST',CLASS=A,MSGCLASS=X
//STEP1   EXEC PGM=PROG1
//INPUT   DD DSN=SOURCE.DATA,DISP=NEW
//OUTPUT  DD DSN=RESULT.DATA,DISP=SHR
//TEMP    DD DSN=WORK.FILE,DISP=(OLD,CATLG)
```

**Solutions :**

1. **INPUT** : `DISP=NEW` mais SOURCE.DATA existe probablement  
   → Devrait être `DISP=SHR` (lecture)

2. **OUTPUT** : `DISP=SHR` mais on écrit dedans  
   → Devrait être `DISP=(NEW,CATLG)` ou `DISP=OLD`

3. **TEMP** : `DISP=(OLD,CATLG)` mais c'est un fichier de travail  
   → Devrait être `DISP=(NEW,DELETE)` ou `DISP=(NEW,PASS)`

**✅ CORRECT :**

```jcl
//TESTJOB JOB (ACCT),'TEST',CLASS=A,MSGCLASS=X
//STEP1   EXEC PGM=PROG1
//INPUT   DD DSN=SOURCE.DATA,DISP=SHR
//OUTPUT  DD DSN=RESULT.DATA,DISP=(NEW,CATLG,DELETE)
//TEMP    DD DSN=&&WORK,DISP=(NEW,PASS)
```

---

## 8. Organisation des Datasets

### Les 3 Types Principaux de Datasets

**Mainframe offre 3 façons d'organiser les données :**

1. **Sequential (PS)** - Fichier séquentiel simple
2. **Partitioned (PDS/PDSE)** - "Bibliothèque" avec des membres
3. **VSAM** - Virtual Storage Access Method (avancé)

### Sequential Datasets (PS) - Approfondissement

**PS = Physical Sequential**

**Structure :**
```
Dataset MY.DATA.FILE
  ├─ Record 1
  ├─ Record 2
  ├─ Record 3
  ├─ ...
  └─ Record N
```

**Caractéristiques :**
- Accès **séquentiel** uniquement (du début à la fin)
- Pas d'index
- Simple et efficace
- Le plus courant pour les données

**Analogie : Une cassette audio VHS**
- Tu dois dérouler pour lire
- Tu ne peux pas sauter directement au milieu

**Quand utiliser PS :**
- Données transactionnelles (logs, transactions bancaires)
- Fichiers de données à traiter en batch
- Exports/imports de données
- Rapports

**Création d'un PS :**

```jcl
//STEP1   EXEC PGM=IEFBR14
//PS      DD DSN=MY.SEQUENTIAL.FILE,
//           DISP=(NEW,CATLG),
//           UNIT=SYSDA,
//           SPACE=(CYL,(10,2),RLSE),
//           DCB=(RECFM=FB,LRECL=80,BLKSIZE=27920)
```

### Partitioned Datasets (PDS)

**PDS = Partitioned Data Set**

**C'est comme un DOSSIER Windows/Linux qui contient des FICHIERS.**

**Structure :**
```
Dataset MY.PDS.LIBRARY
  ├─ Directory (index)
  ├─ Member PROG1
  ├─ Member PROG2
  ├─ Member PROG3
  └─ ...
```

**Vocabulaire :**
- **PDS** = La bibliothèque (le dossier)
- **Member** = Un fichier dans la bibliothèque
- **Directory** = Index des membres

**Analogie : Un classeur avec des onglets**
- Le classeur = PDS
- Chaque onglet = Member
- La table des matières = Directory

**Quand utiliser PDS :**
- Programmes COBOL compilés (load modules)
- Code source (JCL, COBOL, etc.)
- Procedures réutilisables
- Toute collection de "fichiers" similaires

**Exemples réels :**
```
PROD.COBOL.SOURCE   ← PDS de programmes COBOL
  ├─ PAYROLL
  ├─ INVOICE
  └─ REPORT

PROD.LOAD.LIB       ← PDS de programmes compilés
  ├─ PAYROLL
  ├─ INVOICE
  └─ REPORT

PROD.JCL.LIBRARY    ← PDS de jobs JCL
  ├─ DAILY
  ├─ WEEKLY
  └─ MONTHLY
```

### Notation PDS et Member

**Format :**
```
DSN=pds-name(member-name)
```

**Exemples :**

```jcl
//* Référencer tout le PDS
//LIB DD DSN=MY.PDS.LIBRARY,DISP=SHR

//* Référencer UN member spécifique
//PROG DD DSN=MY.PDS.LIBRARY(PROG1),DISP=SHR

//* Référencer plusieurs members
//LIB1 DD DSN=MY.PDS.LIBRARY(PROG1),DISP=SHR
//LIB2 DD DSN=MY.PDS.LIBRARY(PROG2),DISP=SHR
```

### Création d'un PDS

**Un PDS nécessite des DIRECTORY BLOCKS.**

```jcl
//STEP1   EXEC PGM=IEFBR14
//PDS     DD DSN=MY.SOURCE.LIB,
//           DISP=(NEW,CATLG),
//           UNIT=SYSDA,
//           SPACE=(CYL,(10,2,20),RLSE),
//           DCB=(RECFM=FB,LRECL=80,BLKSIZE=27920,DSORG=PO)
                                                    ↑      ↑
                                                    │      └─ DSORG=PO (Partitioned Organization)
                                                    └──────── 20 directory blocks
```

**SPACE pour PDS :**

```jcl
SPACE=(CYL,(primary,secondary,directory-blocks))
              ↑        ↑          ↑
              │        │          └─ Pour l'index des membres
              │        └──────────── Allocations secondaires
              └───────────────────── Allocation primaire
```

**Combien de directory blocks ?**

**Règle approximative :**
- 1 directory block ≈ 6 membres
- Pour 30 membres → 5 directory blocks
- Pour 100 membres → 17 directory blocks

**💡 Surestime toujours un peu. Mieux avoir trop que pas assez.**

**Si tu manques de directory blocks :**
```
DIRECTORY FULL - MEMBER NOT ADDED
```

### PDS vs PDSE

**PDSE = Partitioned Data Set Extended (version moderne)**

**Différences :**

| Caractéristique | PDS | PDSE |
|-----------------|-----|------|
| Directory blocks | Limité, fixe | Dynamique, pas de limite |
| Compression | Manuelle (IEBCOPY) | Automatique |
| Members supprimés | Laissent des "trous" | Space récupéré automatiquement |
| Performance | Bonne | Meilleure |
| Compatibilité | Universelle | z/OS moderne |

**💡 RECOMMANDATION :**

**Utilise PDSE pour les nouvelles bibliothèques.**

**Création d'un PDSE :**

```jcl
//STEP1   EXEC PGM=IEFBR14
//PDSE    DD DSN=MY.SOURCE.LIB,
//           DISP=(NEW,CATLG),
//           UNIT=SYSDA,
//           SPACE=(CYL,(10,2),RLSE),
//           DSNTYPE=LIBRARY,
//           DCB=(RECFM=FB,LRECL=80,BLKSIZE=27920)
               ↑
               DSNTYPE=LIBRARY crée un PDSE (pas besoin de directory blocks)
```

### Choisir le Type de Dataset

**Décision tree :**

```
Besoin d'accès direct (par clé) ?
├─ OUI → VSAM KSDS
└─ NON
    └─ Plusieurs "fichiers" similaires à grouper ?
        ├─ OUI → PDS/PDSE
        └─ NON → Sequential (PS)
```

**Exemples pratiques :**

| Besoin | Type | Exemple |
|--------|------|---------|
| Logs d'application | PS | DAILY.APPLICATION.LOG |
| Transactions bancaires | PS | DAILY.TRANSACTIONS |
| Fichier master clients | VSAM KSDS | PROD.CUSTOMER.MASTER |
| Programmes COBOL source | PDS | PROD.COBOL.SOURCE |
| Programmes compilés | PDS | PROD.LOAD.LIBRARY |
| JCL procedures | PDS | PROD.PROCLIB |

---

## 9. GDG - Generation Data Groups

### Qu'est-ce qu'un GDG ?

**GDG = Generation Data Group**

**C'est un système de VERSIONING pour les datasets.**

**Analogie : Un classeur avec des versions datées**

```
Rapport Mensuel
  ├─ Janvier 2025
  ├─ Février 2025
  ├─ Mars 2025
  └─ ...
```

**Chaque version s'appelle une GENERATION.**

### Pourquoi les GDG sont ESSENTIELS ?

**Problème sans GDG :**

Tu as un job qui tourne tous les jours et crée un rapport :

```jcl
//REPORT DD DSN=DAILY.REPORT,DISP=(NEW,CATLG)
```

**Jour 1 :** Crée `DAILY.REPORT` ✅  
**Jour 2 :** Erreur "DUPLICATE DATASET NAME" ❌

**Solution manuelle (horrible) :**

```jcl
//REPORT DD DSN=DAILY.REPORT.20250115,DISP=(NEW,CATLG)
//REPORT DD DSN=DAILY.REPORT.20250116,DISP=(NEW,CATLG)
//REPORT DD DSN=DAILY.REPORT.20250117,DISP=(NEW,CATLG)
```

Tu dois changer le JCL CHAQUE JOUR.

**Solution GDG (brillante) :**

```jcl
//REPORT DD DSN=DAILY.REPORT(+1),DISP=(NEW,CATLG)
```

**Ça fonctionne TOUS LES JOURS automatiquement !**

### Structure d'un GDG

**Un GDG a :**
1. **GDG Base** - Le nom du groupe (catalogue)
2. **Generations** - Les versions individuelles

**Exemple :**

```
GDG Base: MONTHLY.BACKUP
  ├─ Generation G0001V00 (la plus ancienne)
  ├─ Generation G0002V00
  ├─ Generation G0003V00
  ├─ Generation G0004V00
  └─ Generation G0005V00 (la plus récente)
```

### Notation Relative des GDG

**Tu références les generations de façon RELATIVE :**

```jcl
DSN=gdg-base(relative-generation-number)
```

| Notation | Signification |
|----------|---------------|
| `(+1)` | Nouvelle generation (à créer) |
| `(0)` | Generation la plus récente |
| `(-1)` | Avant-dernière generation |
| `(-2)` | Antépénultième generation |
| `(-n)` | n generations en arrière |

**Exemples :**

```jcl
//* Créer une nouvelle generation
//NEW DD DSN=BACKUP.MASTER(+1),DISP=(NEW,CATLG)

//* Lire la plus récente
//LAST DD DSN=BACKUP.MASTER(0),DISP=SHR

//* Lire celle d'hier
//PREV DD DSN=BACKUP.MASTER(-1),DISP=SHR

//* Comparer aujourd'hui vs hier
//TODAY DD DSN=BACKUP.MASTER(0),DISP=SHR
//YSTRDY DD DSN=BACKUP.MASTER(-1),DISP=SHR
```

### Création d'une GDG Base

**Utilise IDCAMS pour créer la base GDG :**

```jcl
//STEP1  EXEC PGM=IDCAMS
//SYSPRINT DD SYSOUT=*
//SYSIN    DD *
  DEFINE GDG (NAME(DAILY.BACKUP) -
              LIMIT(7) -
              SCRATCH -
              NOEMPTY)
/*
```

**Paramètres :**

**LIMIT(n)** - Nombre maximum de generations à garder

```jcl
LIMIT(7)  ← Garde les 7 dernières generations
```

Quand tu crées la 8ème, la plus ancienne est automatiquement supprimée.

**SCRATCH vs NOSCRATCH**

```jcl
SCRATCH    ← Supprime physiquement les anciennes generations
NOSCRATCH  ← Décatalogue mais garde les données
```

**💡 Utilise toujours SCRATCH (libère l'espace disque).**

**EMPTY vs NOEMPTY**

```jcl
EMPTY    ← Quand LIMIT atteint, supprime TOUTES les generations
NOEMPTY  ← Supprime seulement la plus ancienne (rolling)
```

**💡 Utilise NOEMPTY (comportement normal).**

### Exemple Complet de Setup GDG

```jcl
//GDGSETUP JOB (ACCT),'CREATE GDG',CLASS=A,MSGCLASS=X
//*
//* Create GDG base for daily backups (keep 7 days)
//*
//STEP1  EXEC PGM=IDCAMS
//SYSPRINT DD SYSOUT=*
//SYSIN    DD *
  DEFINE GDG (NAME(BACKUP.DAILY.MASTER) -
              LIMIT(7) -
              SCRATCH -
              NOEMPTY)
/*
//*
//* Create GDG base for monthly reports (keep 12 months)
//*
//STEP2  EXEC PGM=IDCAMS
//SYSPRINT DD SYSOUT=*
//SYSIN    DD *
  DEFINE GDG (NAME(REPORT.MONTHLY.SALES) -
              LIMIT(12) -
              SCRATCH -
              NOEMPTY)
/*
```

### Utilisation d'un GDG dans un Job

**Job de backup quotidien :**

```jcl
//BACKUP  JOB (ACCT),'DAILY BACKUP',CLASS=A,MSGCLASS=X
//*
//* Create today's backup
//*
//STEP1   EXEC PGM=IEBGENER
//SYSPRINT DD SYSOUT=*
//SYSIN    DD DUMMY
//SYSUT1   DD DSN=PROD.MASTER.FILE,DISP=SHR
//SYSUT2   DD DSN=BACKUP.DAILY.MASTER(+1),
//            DISP=(NEW,CATLG),
//            UNIT=SYSDA,
//            SPACE=(CYL,(50,10),RLSE),
//            DCB=(RECFM=FB,LRECL=100,BLKSIZE=27900)
```

**Ce job :**
- Peut tourner TOUS LES JOURS sans modification
- Crée automatiquement une nouvelle generation
- Supprime automatiquement la generation de plus de 7 jours

**Magique !** ✨

### GDG Best Practices

**1. Choisis LIMIT selon la retention nécessaire**

```jcl
LIMIT(7)   ← Backups quotidiens (1 semaine)
LIMIT(30)  ← Logs quotidiens (1 mois)
LIMIT(12)  ← Rapports mensuels (1 an)
LIMIT(52)  ← Rapports hebdomadaires (1 an)
```

**2. Toujours utiliser SCRATCH**

Libère l'espace disque automatiquement.

**3. Toujours utiliser NOEMPTY**

Comportement rolling (supprime la plus ancienne).

**4. Nommage cohérent**

```jcl
BACKUP.DAILY.xxxx    ← Backups quotidiens
BACKUP.WEEKLY.xxxx   ← Backups hebdomadaires
REPORT.MONTHLY.xxxx  ← Rapports mensuels
LOG.DAILY.xxxx       ← Logs quotidiens
```

### Exercice Pratique : GDG

**Crée un système de GDG pour une application bancaire :**

1. GDG pour les transactions quotidiennes (garde 30 jours)
2. GDG pour les backups hebdomadaires (garde 8 semaines)
3. Job qui copie les transactions du jour dans le GDG

**Solution :**

```jcl
//GDGSETUP JOB (ACCT),'BANKING GDG SETUP',CLASS=A,MSGCLASS=X
//*
//* Create daily transactions GDG (30 days retention)
//*
//STEP1  EXEC PGM=IDCAMS
//SYSPRINT DD SYSOUT=*
//SYSIN    DD *
  DEFINE GDG (NAME(BANKING.DAILY.TRANSACTIONS) -
              LIMIT(30) -
              SCRATCH -
              NOEMPTY)
/*
//*
//* Create weekly backup GDG (8 weeks retention)
//*
//STEP2  EXEC PGM=IDCAMS
//SYSPRINT DD SYSOUT=*
//SYSIN    DD *
  DEFINE GDG (NAME(BANKING.WEEKLY.BACKUP) -
              LIMIT(8) -
              SCRATCH -
              NOEMPTY)
/*
```

```jcl
//DAILYTXN JOB (ACCT),'DAILY TXN ARCHIVE',CLASS=A,MSGCLASS=X
//*
//* Archive today's transactions to GDG
//*
//STEP1   EXEC PGM=IEBGENER
//SYSPRINT DD SYSOUT=*
//SYSIN    DD DUMMY
//SYSUT1   DD DSN=BANKING.TODAY.TRANSACTIONS,DISP=SHR
//SYSUT2   DD DSN=BANKING.DAILY.TRANSACTIONS(+1),
//            DISP=(NEW,CATLG),
//            UNIT=SYSDA,
//            SPACE=(CYL,(100,20),RLSE),
//            DCB=(RECFM=FB,LRECL=200,BLKSIZE=27800)
```

---

**📚 FIN DE LA PARTIE 2A**

**Tu maîtrises maintenant :**
✅ DISP en profondeur (toutes les combinaisons)
✅ Organisation des datasets (PS, PDS, PDSE)
✅ GDG (versioning automatique)

**Passe à la Partie 2B pour :**
- Catalogues et Volumes
- IEBGENER
- IEBCOPY

---

**💎 100% Gratuit • Pour Tous • À Jamais**  
**🔗 GitHub : Learning Schooling Foundation**
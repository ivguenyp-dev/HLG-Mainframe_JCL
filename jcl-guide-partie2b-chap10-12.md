# JCL - Job Control Language : Guide Complet Mondial
## Partie 2B : Catalogues, IEBGENER, IEBCOPY (Chapitres 10-12)

**🔗 Repository GitHub :** [Learning Schooling Foundation - JCL](https://github.com/learning-schooling-foundation)

---

## Table des Matières - Partie 2B

10. [Catalogues et Volumes](#10-catalogues-et-volumes)
11. [IEBGENER - Copier des Données](#11-iebgener---copier-des-données)
12. [IEBCOPY - Gérer les PDS](#12-iebcopy---gérer-les-pds)

---

## 10. Catalogues et Volumes

### Qu'est-ce qu'un Catalogue ?

**Le CATALOGUE est l'index de tous les datasets sur le mainframe.**

**Analogie : Le catalogue d'une bibliothèque**
- Chaque livre (dataset) a une fiche
- La fiche indique sur quelle étagère (volume) se trouve le livre
- Tu ne cherches pas le livre sur chaque étagère, tu consultes le catalogue

**Le système catalogue maintient :**
- Nom du dataset
- Type (PS, PDS, VSAM, etc.)
- Volume (disque) où il se trouve
- Caractéristiques (DCB, etc.)
- Date de création
- Date de dernier accès

### Pourquoi Cataloguer ?

**Avec catalogue :**

```jcl
//INPUT DD DSN=MY.DATA.FILE,DISP=SHR
```

Le système :
1. Consulte le catalogue
2. Trouve que MY.DATA.FILE est sur le volume WORK01
3. L'ouvre

**Sans catalogue :**

```jcl
//INPUT DD DSN=MY.DATA.FILE,
//         DISP=SHR,
//         VOL=SER=WORK01
          ↑
          Tu DOIS spécifier le volume manuellement
```

**💡 Cataloguer = Simplicité + Flexibilité**

Le fichier peut être déplacé vers un autre volume, le catalogue est mis à jour, mais ton JCL ne change pas.

### DISP et le Catalogue

**Rappel des dispositions qui cataloguent :**

```jcl
DISP=(NEW,CATLG)      ← Catalogue à la création
DISP=(OLD,CATLG)      ← Catalogue un fichier existant non catalogué
DISP=(NEW,CATLG,DELETE) ← Catalogue si OK, supprime si erreur
```

**Dispositions qui décataloguent :**

```jcl
DISP=(OLD,UNCATLG)    ← Décatalogue (garde le fichier)
DISP=(OLD,DELETE)     ← Supprime du catalogue ET du disque
```

### Master Catalog vs User Catalog

**Mainframe a deux types de catalogues :**

**1. Master Catalog**
- UN seul par système
- Catalogue "racine"
- Contient les datasets système critiques
- Contient les références vers les user catalogs

**2. User Catalog(s)**
- Plusieurs sur un système
- Catalogues "utilisateur"
- Contient les datasets des applications

**Analogie : Catalogue de bibliothèque**
- Master Catalog = Catalogue principal de la ville (référence toutes les bibliothèques)
- User Catalog = Catalogue d'une bibliothèque spécifique

**En pratique, tu n'as presque jamais besoin de t'en préoccuper.**

Le système gère ça automatiquement selon le nom du dataset.

### High-Level Qualifier (HLQ)

**Le premier qualifier d'un dataset détermine le catalogue.**

```jcl
DSN=PROD.APPLICATION.DATA
    ↑
    High-Level Qualifier (HLQ)
```

**L'installation définit des règles :**

```
PROD.*    → User Catalog PRODCAT
TEST.*    → User Catalog TESTCAT
DEV.*     → User Catalog DEVCAT
SYS1.*    → Master Catalog
```

**💡 Pour toi, c'est transparent. Utilise juste le bon HLQ selon l'environnement.**

### VOL=SER - Spécifier le Volume

**Si un dataset N'EST PAS catalogué, tu DOIS spécifier le volume :**

```jcl
//INPUT DD DSN=UNCATALOGED.FILE,
//         DISP=SHR,
//         VOL=SER=WORK01
```

**VOL=SER=xxxxxx** où xxxxxx est le nom du volume (6 caractères max).

**Exemples de noms de volumes :**
```
WORK01
PROD99
USER02
SYSRES
```

### Cataloguer un Dataset Existant

**Scénario : Tu as un dataset non catalogué et tu veux le cataloguer.**

**Méthode 1 : IEFBR14**

```jcl
//STEP1 EXEC PGM=IEFBR14
//FILE  DD DSN=UNCATALOGED.FILE,
//         DISP=(OLD,CATLG),
//         VOL=SER=WORK01
```

**Méthode 2 : IDCAMS**

```jcl
//STEP1  EXEC PGM=IDCAMS
//SYSPRINT DD SYSOUT=*
//SYSIN    DD *
  DEFINE NONVSAM (NAME(UNCATALOGED.FILE) -
                  VOLUMES(WORK01) -
                  DEVT(3390))
/*
```

### Décataloguer un Dataset

**Décataloguer = Retirer du catalogue SANS supprimer les données**

```jcl
//STEP1 EXEC PGM=IEFBR14
//FILE  DD DSN=MY.FILE,
//         DISP=(OLD,UNCATLG)
```

**Ou avec IDCAMS :**

```jcl
//STEP1  EXEC PGM=IDCAMS
//SYSPRINT DD SYSOUT=*
//SYSIN    DD *
  DELETE MY.FILE NOSCRATCH
/*
```

`NOSCRATCH` = Décatalogue mais garde les données.

### Lister les Datasets Catalogués

**Lister tous les datasets commençant par un HLQ :**

```jcl
//LISTCAT EXEC PGM=IDCAMS
//SYSPRINT DD SYSOUT=*
//SYSIN    DD *
  LISTCAT LEVEL(PROD) VOLUME
/*
```

**Lister un dataset spécifique :**

```jcl
//LISTCAT EXEC PGM=IDCAMS
//SYSPRINT DD SYSOUT=*
//SYSIN    DD *
  LISTCAT ENTRIES(PROD.MASTER.FILE) ALL
/*
```

**Sortie typique :**

```
NONVSAM ------- PROD.MASTER.FILE
  IN-CAT --- CATALOG.USER.PROD
  HISTORY
    CREATION---2025-01-15  EXPIRATION---0000-00-00
  VOLSER------------PROD01
  DEVTYP------------X'3010200F' (3390)
  ASSOCIATIONS
    NONVSAM--------(NULL)
```

### Migrated Datasets (HSM)

**HSM = Hierarchical Storage Management**

**Les datasets peu utilisés sont automatiquement MIGRÉS vers tape ou disque secondaire pour économiser l'espace.**

**Si tu essaies d'accéder à un dataset migré :**

```jcl
//INPUT DD DSN=OLD.FILE,DISP=SHR
```

**Erreur :**
```
DATASET MIGRATED
```

**Le système peut le RAPPELER automatiquement (ou tu dois le faire manuellement).**

**Rappel manuel :**

```jcl
//RECALL EXEC PGM=IEFBR14
//FILE   DD DSN=OLD.FILE,DISP=SHR
```

Ou commande TSO : `HRECALL 'OLD.FILE'`

**💡 Si tu vois "MIGRATED", ne panique pas. C'est normal pour les vieux fichiers.**

### Exercice Pratique : Catalogues

**Tu as un dataset non catalogué `OLD.DATA.FILE` sur le volume `WORK05`.**

1. Catalogue-le
2. Copie-le vers un nouveau dataset `NEW.DATA.FILE`
3. Décatalogue l'ancien (garde les données)

**Solution :**

```jcl
//EXERCISE JOB (ACCT),'CATALOG EXERCISE',CLASS=A,MSGCLASS=X
//*
//* STEP1: Catalog the existing dataset
//*
//STEP1   EXEC PGM=IEFBR14
//OLD     DD DSN=OLD.DATA.FILE,
//           DISP=(OLD,CATLG),
//           VOL=SER=WORK05
//*
//* STEP2: Copy to new cataloged dataset
//*
//STEP2   EXEC PGM=IEBGENER
//SYSPRINT DD SYSOUT=*
//SYSIN    DD DUMMY
//SYSUT1   DD DSN=OLD.DATA.FILE,DISP=SHR
//SYSUT2   DD DSN=NEW.DATA.FILE,
//            DISP=(NEW,CATLG),
//            UNIT=SYSDA,
//            SPACE=(CYL,(10,2),RLSE)
//*
//* STEP3: Uncatalog the old dataset (keep data)
//*
//STEP3   EXEC PGM=IEFBR14
//OLD     DD DSN=OLD.DATA.FILE,
//           DISP=(OLD,UNCATLG)
```

---

## 11. IEBGENER - Copier des Données

### Qu'est-ce qu'IEBGENER ?

**IEBGENER = Utilitaire IBM de copie séquentielle**

**C'est L'utilitaire le plus utilisé sur mainframe pour :**
- Copier un dataset vers un autre
- Copier un member de PDS vers un PS
- Copier des données instream vers un dataset
- Convertir des formats (avec édition basique)

**Analogie : La commande `cp` de Linux ou `copy` de Windows**

### Structure d'un Job IEBGENER

```jcl
//stepname EXEC PGM=IEBGENER
//SYSPRINT DD SYSOUT=*
//SYSIN    DD DUMMY (ou DD *)
//SYSUT1   DD DSN=source,DISP=...
//SYSUT2   DD DSN=destination,DISP=...
```

**DDnames obligatoires :**
- **SYSPRINT** - Messages de sortie
- **SYSIN** - Commandes de contrôle (ou DUMMY si aucune)
- **SYSUT1** - Fichier source (entrée)
- **SYSUT2** - Fichier destination (sortie)

### Copie Simple - Exemple de Base

```jcl
//COPY    JOB (ACCT),'SIMPLE COPY',CLASS=A,MSGCLASS=X
//STEP1   EXEC PGM=IEBGENER
//SYSPRINT DD SYSOUT=*
//SYSIN    DD DUMMY
//SYSUT1   DD DSN=SOURCE.DATA,DISP=SHR
//SYSUT2   DD DSN=DEST.DATA,
//            DISP=(NEW,CATLG),
//            UNIT=SYSDA,
//            SPACE=(CYL,(10,2),RLSE),
//            DCB=(RECFM=FB,LRECL=80,BLKSIZE=27920)
```

**Ce qui se passe :**
1. Lit tous les records de SOURCE.DATA
2. Les copie dans DEST.DATA
3. Ferme les fichiers

**Simple, efficace, fiable.**

### Copier d'un PDS Member vers un PS

```jcl
//STEP1   EXEC PGM=IEBGENER
//SYSPRINT DD SYSOUT=*
//SYSIN    DD DUMMY
//SYSUT1   DD DSN=SOURCE.PDS(MEMBER1),DISP=SHR
//SYSUT2   DD DSN=DEST.PS.FILE,
//            DISP=(NEW,CATLG),
//            SPACE=(TRK,(10,5))
```

### Copier d'un PS vers un PDS Member

```jcl
//STEP1   EXEC PGM=IEBGENER
//SYSPRINT DD SYSOUT=*
//SYSIN    DD DUMMY
//SYSUT1   DD DSN=SOURCE.PS.FILE,DISP=SHR
//SYSUT2   DD DSN=DEST.PDS(NEWMEM),DISP=SHR
```

**Si NEWMEM existe, il est remplacé.**  
**Si NEWMEM n'existe pas, il est créé.**

### Copier Plusieurs Fichiers (Concatenation)

**Combiner plusieurs sources en une destination :**

```jcl
//STEP1   EXEC PGM=IEBGENER
//SYSPRINT DD SYSOUT=*
//SYSIN    DD DUMMY
//SYSUT1   DD DSN=FILE1,DISP=SHR
//         DD DSN=FILE2,DISP=SHR
//         DD DSN=FILE3,DISP=SHR
//SYSUT2   DD DSN=COMBINED.FILE,
//            DISP=(NEW,CATLG),
//            SPACE=(CYL,(10,2))
```

**IEBGENER lit FILE1, puis FILE2, puis FILE3 et les écrit dans COMBINED.FILE.**

### Copier des Données Instream

**Créer un dataset à partir de données dans le JCL :**

```jcl
//STEP1   EXEC PGM=IEBGENER
//SYSPRINT DD SYSOUT=*
//SYSIN    DD DUMMY
//SYSUT1   DD *
RECORD 1 DATA HERE
RECORD 2 DATA HERE
RECORD 3 DATA HERE
/*
//SYSUT2   DD DSN=OUTPUT.FILE,
//            DISP=(NEW,CATLG),
//            UNIT=SYSDA,
//            SPACE=(TRK,(1,1)),
//            DCB=(RECFM=FB,LRECL=80,BLKSIZE=800)
```

**Utile pour créer de petits fichiers de test.**

### Copier vers SYSOUT (Imprimer)

```jcl
//STEP1   EXEC PGM=IEBGENER
//SYSPRINT DD SYSOUT=*
//SYSIN    DD DUMMY
//SYSUT1   DD DSN=SOURCE.DATA,DISP=SHR
//SYSUT2   DD SYSOUT=*
```

**Le contenu de SOURCE.DATA est envoyé vers le spool (consultable en ligne ou imprimable).**

### IEBGENER vs ICEGENER

**ICEGENER est une version moderne et plus rapide d'IEBGENER.**

```jcl
//STEP1   EXEC PGM=ICEGENER
```

**Même syntaxe, même DDnames, mais :**
- Plus rapide
- Gère de plus gros fichiers
- Plus d'options

**💡 Si ton installation a ICEGENER, utilise-le à la place d'IEBGENER.**

### Return Codes IEBGENER

**IEBGENER retourne :**

| RC | Signification |
|----|---------------|
| 0 | Succès complet |
| 4 | Warning (traitement OK mais attention) |
| 8 | Erreur (traitement incomplet) |
| 12 | Erreur grave (traitement arrêté) |
| 16 | Erreur fatale |

**💡 RC=0 = Tout va bien**

### Cas d'Usage Réels

**1. Backup quotidien :**

```jcl
//BACKUP  EXEC PGM=IEBGENER
//SYSPRINT DD SYSOUT=*
//SYSIN    DD DUMMY
//SYSUT1   DD DSN=PROD.MASTER.FILE,DISP=SHR
//SYSUT2   DD DSN=BACKUP.MASTER(+1),
//            DISP=(NEW,CATLG),
//            SPACE=(CYL,(50,10))
```

**2. Extraire un member pour révision :**

```jcl
//EXTRACT EXEC PGM=IEBGENER
//SYSPRINT DD SYSOUT=*
//SYSIN    DD DUMMY
//SYSUT1   DD DSN=PROD.COBOL.SOURCE(PAYROLL),DISP=SHR
//SYSUT2   DD SYSOUT=*
```

**3. Créer un fichier de test :**

```jcl
//CREATETST EXEC PGM=IEBGENER
//SYSPRINT DD SYSOUT=*
//SYSIN    DD DUMMY
//SYSUT1   DD *
TEST RECORD 001
TEST RECORD 002
TEST RECORD 003
/*
//SYSUT2   DD DSN=TEST.DATA.FILE,
//            DISP=(NEW,CATLG),
//            SPACE=(TRK,(1,1)),
//            DCB=(RECFM=FB,LRECL=80,BLKSIZE=800)
```

**4. Combiner des logs :**

```jcl
//COMBINE EXEC PGM=IEBGENER
//SYSPRINT DD SYSOUT=*
//SYSIN    DD DUMMY
//SYSUT1   DD DSN=LOG.MONDAY,DISP=SHR
//         DD DSN=LOG.TUESDAY,DISP=SHR
//         DD DSN=LOG.WEDNESDAY,DISP=SHR
//SYSUT2   DD DSN=LOG.WEEKLY,
//            DISP=(NEW,CATLG),
//            SPACE=(CYL,(5,1))
```

### Erreurs Courantes et Solutions

**ERREUR #1 : "SYSUT1 DD STATEMENT MISSING"**

```jcl
//STEP1   EXEC PGM=IEBGENER
//SYSPRINT DD SYSOUT=*
//SYSIN    DD DUMMY
//SYSUT2   DD DSN=OUTPUT,DISP=(NEW,CATLG)
```

**Cause :** Oubli de SYSUT1  
**Solution :** Ajoute `//SYSUT1 DD ...`

---

**ERREUR #2 : "INCOMPATIBLE DCB"**

```jcl
//SYSUT1 DD DSN=INPUT,DISP=SHR,
//          DCB=(RECFM=FB,LRECL=80)
//SYSUT2 DD DSN=OUTPUT,DISP=(NEW,CATLG),
//          DCB=(RECFM=FB,LRECL=100)
```

**Cause :** LRECL différent entre source et destination  
**Solution :** Utilise le même DCB ou laisse IEBGENER copier le DCB automatiquement

---

**ERREUR #3 : "NO SPACE"**

```jcl
//SYSUT2 DD DSN=OUTPUT,
//          DISP=(NEW,CATLG),
//          SPACE=(TRK,(1,0))
```

**Cause :** Pas assez d'espace alloué  
**Solution :** Augmente SPACE

### Exercice Pratique : IEBGENER

**Crée un job qui :**
1. Combine 3 fichiers de transactions (TRANS.MON, TRANS.TUE, TRANS.WED)
2. Les copie dans un GDG (TRANS.WEEKLY)
3. Crée aussi une copie dans SYSOUT pour vérification

**Solution :**

```jcl
//COMBINE JOB (ACCT),'COMBINE TRANS',CLASS=A,MSGCLASS=X
//*
//* Combine 3 daily transaction files into weekly GDG
//*
//STEP1   EXEC PGM=IEBGENER
//SYSPRINT DD SYSOUT=*
//SYSIN    DD DUMMY
//SYSUT1   DD DSN=TRANS.MON,DISP=SHR
//         DD DSN=TRANS.TUE,DISP=SHR
//         DD DSN=TRANS.WED,DISP=SHR
//SYSUT2   DD DSN=TRANS.WEEKLY(+1),
//            DISP=(NEW,CATLG),
//            UNIT=SYSDA,
//            SPACE=(CYL,(10,2),RLSE),
//            DCB=(RECFM=FB,LRECL=100,BLKSIZE=27900)
//*
//* Also copy to SYSOUT for verification
//*
//STEP2   EXEC PGM=IEBGENER
//SYSPRINT DD SYSOUT=*
//SYSIN    DD DUMMY
//SYSUT1   DD DSN=TRANS.WEEKLY(0),DISP=SHR
//SYSUT2   DD SYSOUT=*
```

---

## 12. IEBCOPY - Gérer les PDS

### Qu'est-ce qu'IEBCOPY ?

**IEBCOPY = Utilitaire IBM pour gérer les PDS (Partitioned Data Sets)**

**IEBCOPY peut :**
- Copier un PDS entier
- Copier des members sélectionnés
- Renommer des members
- Compresser un PDS (récupérer l'espace)
- Merger plusieurs PDS
- Créer un backup d'un PDS

**C'est L'outil pour gérer les bibliothèques (load modules, source, JCL, etc.).**

### Structure d'un Job IEBCOPY

```jcl
//stepname EXEC PGM=IEBCOPY
//SYSPRINT DD SYSOUT=*
//indd     DD DSN=input-pds,DISP=SHR
//outdd    DD DSN=output-pds,DISP=...
//SYSUT3   DD UNIT=SYSDA,SPACE=(CYL,(5,1))
//SYSUT4   DD UNIT=SYSDA,SPACE=(CYL,(5,1))
//SYSIN    DD *
  COPY commands here
/*
```

**DDnames :**
- **SYSPRINT** - Messages
- **SYSUTn** - Work datasets (temporaires)
- **DDnames variables** - Input/output PDS (tu choisis les noms)
- **SYSIN** - Commandes IEBCOPY

### Copier un PDS Entier

```jcl
//COPYPDS JOB (ACCT),'COPY PDS',CLASS=A,MSGCLASS=X
//STEP1   EXEC PGM=IEBCOPY
//SYSPRINT DD SYSOUT=*
//INPDS    DD DSN=SOURCE.PDS,DISP=SHR
//OUTPDS   DD DSN=DEST.PDS,
//            DISP=(NEW,CATLG),
//            UNIT=SYSDA,
//            SPACE=(CYL,(10,2,20),RLSE),
//            DCB=(RECFM=FB,LRECL=80,BLKSIZE=27920,DSORG=PO)
//SYSIN    DD *
  COPY INDD=INPDS,OUTDD=OUTPDS
/*
```

**Résultat :** Tous les members de SOURCE.PDS sont copiés dans DEST.PDS

### Copier des Members Sélectionnés

**Copier seulement certains members :**

```jcl
//STEP1   EXEC PGM=IEBCOPY
//SYSPRINT DD SYSOUT=*
//IN       DD DSN=SOURCE.PDS,DISP=SHR
//OUT      DD DSN=DEST.PDS,DISP=SHR
//SYSIN    DD *
  COPY INDD=IN,OUTDD=OUT
  SELECT MEMBER=(PROG1,PROG2,PROG3)
/*
```

**Seuls PROG1, PROG2, et PROG3 sont copiés.**

### Exclure des Members

**Copier tous sauf certains :**

```jcl
//SYSIN    DD *
  COPY INDD=IN,OUTDD=OUT
  EXCLUDE MEMBER=(OLD1,OLD2,OBSOLETE)
/*
```

**Tous les members SAUF OLD1, OLD2, et OBSOLETE sont copiés.**

### Renommer un Member

```jcl
//STEP1   EXEC PGM=IEBCOPY
//SYSPRINT DD SYSOUT=*
//IN       DD DSN=SOURCE.PDS,DISP=SHR
//OUT      DD DSN=DEST.PDS,DISP=SHR
//SYSIN    DD *
  COPY INDD=IN,OUTDD=OUT
  SELECT MEMBER=((OLDNAME,NEWNAME))
/*
```

**Le member OLDNAME est copié sous le nom NEWNAME dans DEST.PDS.**

### Compresser un PDS

**Pourquoi compresser ?**

Quand tu supprimes des members d'un PDS, l'espace n'est pas récupéré. Il reste des "trous".

**Compression = Réorganiser le PDS pour récupérer l'espace.**

```jcl
//COMPRESS JOB (ACCT),'COMPRESS PDS',CLASS=A,MSGCLASS=X
//STEP1   EXEC PGM=IEBCOPY
//SYSPRINT DD SYSOUT=*
//LIB      DD DSN=MY.PDS.LIBRARY,DISP=SHR
//SYSIN    DD *
  COPY INDD=LIB,OUTDD=LIB
/*
```

**Oui, INDD et OUTDD pointent vers le MÊME PDS.**

**Résultat :** Le PDS est compressé in-place.

**💡 Fais ça régulièrement sur les PDS très utilisés.**

### Merger Plusieurs PDS

**Combiner plusieurs PDS en un seul :**

```jcl
//MERGE   JOB (ACCT),'MERGE PDS',CLASS=A,MSGCLASS=X
//STEP1   EXEC PGM=IEBCOPY
//SYSPRINT DD SYSOUT=*
//PDS1     DD DSN=SOURCE.PDS1,DISP=SHR
//PDS2     DD DSN=SOURCE.PDS2,DISP=SHR
//PDS3     DD DSN=SOURCE.PDS3,DISP=SHR
//OUT      DD DSN=COMBINED.PDS,
//            DISP=(NEW,CATLG),
//            SPACE=(CYL,(20,5,50))
//SYSIN    DD *
  COPY INDD=PDS1,OUTDD=OUT
  COPY INDD=PDS2,OUTDD=OUT
  COPY INDD=PDS3,OUTDD=OUT
/*
```

**Tous les members des 3 PDS sont dans COMBINED.PDS.**

**⚠️ Si un member existe dans plusieurs PDS, le dernier copié écrase les précédents.**

### Unloader/Loader (Backup/Restore)

**IEBCOPY peut créer un backup d'un PDS dans un PS (dataset séquentiel).**

**Unload (Backup) :**

```jcl
//UNLOAD  JOB (ACCT),'UNLOAD PDS',CLASS=A,MSGCLASS=X
//STEP1   EXEC PGM=IEBCOPY
//SYSPRINT DD SYSOUT=*
//SOURCE   DD DSN=PROD.COBOL.SOURCE,DISP=SHR
//BACKUP   DD DSN=BACKUP.COBOL.SOURCE,
//            DISP=(NEW,CATLG),
//            UNIT=TAPE,
//            DCB=(RECFM=FB,LRECL=80,BLKSIZE=32720)
//SYSIN    DD *
  COPY OUTDD=BACKUP,INDD=SOURCE
/*
```

**Load (Restore) :**

```jcl
//LOAD    JOB (ACCT),'LOAD PDS',CLASS=A,MSGCLASS=X
//STEP1   EXEC PGM=IEBCOPY
//SYSPRINT DD SYSOUT=*
//BACKUP   DD DSN=BACKUP.COBOL.SOURCE,DISP=SHR
//RESTORED DD DSN=RESTORED.COBOL.SOURCE,
//            DISP=(NEW,CATLG),
//            SPACE=(CYL,(10,2,20))
//SYSIN    DD *
  COPY INDD=BACKUP,OUTDD=RESTORED
/*
```

**💡 Pratique pour backup sur tape ou pour transporter un PDS.**

### REPLACE Option

**Par défaut, si un member existe dans la destination, il N'EST PAS écrasé.**

**Pour forcer le remplacement :**

```jcl
//SYSIN    DD *
  COPY INDD=IN,OUTDD=OUT
  SELECT MEMBER=(PROG1,PROG2)
  REPLACE
/*
```

**Maintenant PROG1 et PROG2 écrasent les versions existantes dans OUT.**

### Cas d'Usage Réels

**1. Backup hebdomadaire des programmes source :**

```jcl
//BACKUP  JOB (ACCT),'WEEKLY SOURCE BACKUP',CLASS=A,MSGCLASS=X
//STEP1   EXEC PGM=IEBCOPY
//SYSPRINT DD SYSOUT=*
//SOURCE   DD DSN=PROD.COBOL.SOURCE,DISP=SHR
//BACKUP   DD DSN=BACKUP.SOURCE.WEEKLY(+1),
//            DISP=(NEW,CATLG),
//            SPACE=(CYL,(50,10))
//SYSIN    DD *
  COPY INDD=SOURCE,OUTDD=BACKUP
/*
```

**2. Promouvoir du TEST vers PROD :**

```jcl
//PROMOTE JOB (ACCT),'PROMOTE TO PROD',CLASS=A,MSGCLASS=X
//STEP1   EXEC PGM=IEBCOPY
//SYSPRINT DD SYSOUT=*
//TEST     DD DSN=TEST.LOAD.LIB,DISP=SHR
//PROD     DD DSN=PROD.LOAD.LIB,DISP=SHR
//SYSIN    DD *
  COPY INDD=TEST,OUTDD=PROD
  SELECT MEMBER=(PAYROLL,INVOICE,REPORT)
  REPLACE
/*
```

**3. Compression mensuelle des bibliothèques :**

```jcl
//COMPRESS JOB (ACCT),'MONTHLY COMPRESS',CLASS=A,MSGCLASS=X
//STEP1   EXEC PGM=IEBCOPY
//SYSPRINT DD SYSOUT=*
//LIB1     DD DSN=PROD.COBOL.SOURCE,DISP=SHR
//SYSIN    DD *
  COPY INDD=LIB1,OUTDD=LIB1
/*
//STEP2   EXEC PGM=IEBCOPY
//SYSPRINT DD SYSOUT=*
//LIB2     DD DSN=PROD.LOAD.LIB,DISP=SHR
//SYSIN    DD *
  COPY INDD=LIB2,OUTDD=LIB2
/*
```

### Return Codes IEBCOPY

| RC | Signification |
|----|---------------|
| 0 | Succès |
| 4 | Warning (OK mais vérifier messages) |
| 8 | Erreur (copie partielle) |
| 12 | Erreur grave |
| 16 | Erreur fatale |

### Erreurs Courantes

**ERREUR #1 : "DIRECTORY FULL"**

```
IEB1014I DIRECTORY IS FULL
```

**Cause :** Pas assez de directory blocks dans le PDS destination  
**Solution :** Crée le PDS avec plus de directory blocks

---

**ERREUR #2 : "MEMBER NOT FOUND"**

```
IEB1015I MEMBER xxx NOT FOUND
```

**Cause :** Le member spécifié n'existe pas dans le PDS source  
**Solution :** Vérifie le nom du member

### Exercice Pratique : IEBCOPY

**Tu dois :**
1. Créer un PDS de test avec 3 members
2. Copier seulement 2 members vers un nouveau PDS
3. Compresser le PDS original

**Solution :**

```jcl
//EXERCISE JOB (ACCT),'IEBCOPY EXERCISE',CLASS=A,MSGCLASS=X
//*
//* STEP1-3: Create test PDS with 3 members
//*
//STEP1   EXEC PGM=IEBGENER
//SYSPRINT DD SYSOUT=*
//SYSIN    DD DUMMY
//SYSUT1   DD *
MEMBER 1 DATA
/*
//SYSUT2   DD DSN=TEST.PDS(MEM1),
//            DISP=(NEW,CATLG),
//            SPACE=(TRK,(5,1,5)),
//            DCB=(RECFM=FB,LRECL=80,BLKSIZE=800,DSORG=PO)
//STEP2   EXEC PGM=IEBGENER
//SYSPRINT DD SYSOUT=*
//SYSIN    DD DUMMY
//SYSUT1   DD *
MEMBER 2 DATA
/*
//SYSUT2   DD DSN=TEST.PDS(MEM2),DISP=SHR
//STEP3   EXEC PGM=IEBGENER
//SYSPRINT DD SYSOUT=*
//SYSIN    DD DUMMY
//SYSUT1   DD *
MEMBER 3 DATA
/*
//SYSUT2   DD DSN=TEST.PDS(MEM3),DISP=SHR
//*
//* STEP4: Copy only MEM1 and MEM2 to new PDS
//*
//STEP4   EXEC PGM=IEBCOPY
//SYSPRINT DD SYSOUT=*
//SOURCE   DD DSN=TEST.PDS,DISP=SHR
//DEST     DD DSN=TEST.SUBSET.PDS,
//            DISP=(NEW,CATLG),
//            SPACE=(TRK,(5,1,5))
//SYSIN    DD *
  COPY INDD=SOURCE,OUTDD=DEST
  SELECT MEMBER=(MEM1,MEM2)
/*
//*
//* STEP5: Compress original PDS
//*
//STEP5   EXEC PGM=IEBCOPY
//SYSPRINT DD SYSOUT=*
//LIB      DD DSN=TEST.PDS,DISP=SHR
//SYSIN    DD *
  COPY INDD=LIB,OUTDD=LIB
/*
```

---

## Conclusion de la Partie 2

**🎉 FÉLICITATIONS ! Tu as terminé TOUTE la Partie 2 !**

### Ce que tu maîtrises maintenant

**Gestion Avancée des Datasets :**
✅ **DISP en profondeur** - Toutes les combinaisons, erreurs classiques, best practices  
✅ **Organisation** - PS, PDS, PDSE, VSAM, quand utiliser quoi  
✅ **GDG** - Versioning automatique, backups, rapports  
✅ **Catalogues** - Comment retrouver et gérer les datasets  
✅ **IEBGENER** - Copier n'importe quoi vers n'importe quoi  
✅ **IEBCOPY** - Gérer les PDS comme un pro  

### Tu es capable de :

🚀 **Gérer le cycle de vie** complet des datasets  
🚀 **Créer des structures** de données professionnelles  
🚀 **Automatiser les backups** avec GDG  
🚀 **Copier et gérer** des bibliothèques entières  
🚀 **Débugger** les problèmes de datasets  
🚀 **Optimiser** l'utilisation de l'espace disque  

### Prochaine Étape : Partie 3

**La Partie 3 couvrira :**

13. **SORT/DFSORT** - Trier et manipuler les données (L'utilitaire le plus puissant)  
14. **IDCAMS - L'Outil VSAM** - Gérer les fichiers VSAM  
15. **Procedures (PROCS)** - Créer du JCL réutilisable  
16. **Paramètres Symboliques** - Rendre les PROCS flexibles  
17. **Conditional Processing** - IF/THEN/ELSE en JCL  
18. **JES2/JES3** - Job Entry Subsystem avancé  

### Tu as Fait un ÉNORME Chemin

**Avec ce que tu sais maintenant, tu peux déjà :**
- Travailler sur du JCL de production
- Gérer les datasets d'une application complète
- Créer des jobs de backup/restore automatisés
- Maintenir des bibliothèques de programmes
- Comprendre et modifier n'importe quel job existant
- **Postuler à des postes junior mainframe avec confiance**

### Le Mindset Mainframe

**Le mainframe est différent.**

Ce n'est pas comme le cloud où tu peux facilement créer/détruire des ressources.

**Sur mainframe :**
- Chaque job doit être **PENSÉ** avant d'être exécuté
- Les erreurs coûtent **CHER** (temps CPU, impact production)
- La **FIABILITÉ** est priorité absolue
- On **PLANIFIE** tout minutieusement

**C'est une discipline différente, mais c'est ce qui rend les mainframers si VALORISÉS.**

### Message pour Toi

**Si tu as lu jusqu'ici, tu fais partie des 1%.**

La plupart abandonnent après la Partie 1.

**Toi, tu as persévéré.**

Tu as appris :
- Les fondations (Partie 1)
- La maîtrise des datasets (Partie 2)

**Tu n'es plus un débutant.**

**Tu es un mainframer en devenir.**

### L'Impact que Tu Vas Avoir

**Imagine :**

Un jeune de Dakar trouve ce guide sur GitHub. Il étudie, pratique, maîtrise le JCL. En 6 mois, il décroche un job remote pour une banque française à 1500€/mois. Dans son quartier, c'est 5× le salaire moyen.

Une mère de Mumbai, bloquée dans un travail manuel, découvre LSF. Elle étudie le soir après le travail. En 1 an, elle devient mainframer junior. Ses enfants vont à l'école privée maintenant.

Un gamin d'une favela de São Paulo trouve ce guide imprimé dans un centre communautaire. Il apprend, il code, il pratique. À 18 ans, il travaille pour Banco do Brasil.

**TOI, en apprenant le JCL avec ce guide, tu fais partie de cette révolution silencieuse.**

---

**Tu es prêt pour la Partie 3 ?**

---

**📚 JCL - Job Control Language : Guide Complet Mondial - Partie 2B**  
**💎 100% Gratuit • Pour Tous • À Jamais**  
**🔗 GitHub : Learning Schooling Foundation**  
**🌍 Pour les kids de Dakar, Mumbai, São Paulo, Gaza, Kinshasa, et partout dans le monde**

**En mémoire d'Aaron Swartz (1986-2013)**  
*"The world is ours to change."*

**Avec l'esprit de Richard Stallman**  
*"Free software is a matter of liberty, not price."*

**Et la détermination de Dan**  
*"Ce savoir n'appartient à personne. Il appartient à l'humanité."*

---
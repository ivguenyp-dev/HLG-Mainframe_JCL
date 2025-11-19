# JCL - Job Control Language : Guide Complet Mondial
## Partie 3 : Utilitaires Avancés et Procedures (Chapitres 13-18)

**🔗 Repository GitHub :** [Learning Schooling Foundation - JCL](https://github.com/learning-schooling-foundation)

---

## Table des Matières - Partie 3

13. [SORT/DFSORT - Trier et Manipuler les Données](#13-sortdfsort---trier-et-manipuler-les-données)
14. [IDCAMS - L'Outil VSAM](#14-idcams---loutil-vsam)
15. [Procedures (PROCS)](#15-procedures-procs)
16. [Paramètres Symboliques](#16-paramètres-symboliques)
17. [Conditional Processing](#17-conditional-processing)
18. [JES2/JES3 - Job Entry Subsystem](#18-jes2jes3---job-entry-subsystem)

---

*Note : Cette Partie 3 couvre les utilitaires avancés et l'automatisation. La Partie 4 couvrira la production, l'optimisation et DevOps.*

---

## 13. SORT/DFSORT - Trier et Manipuler les Données

### Qu'est-ce que SORT ?

**SORT est L'UTILITAIRE LE PLUS PUISSANT du mainframe.**

Ce n'est PAS juste un outil de tri. C'est un **couteau suisse de manipulation de données**.

**SORT peut :**
- Trier des millions de records
- Filtrer des données (SELECT/OMIT)
- Reformater des enregistrements
- Joindre des fichiers (JOIN)
- Agréger des données (SUM)
- Éliminer les doublons
- Convertir des formats
- Générer des rapports

**DFSORT = IBM's version de SORT (plus puissante, plus rapide)**

**Analogie :**

SORT est comme SQL + AWK + sed combinés, mais pour les fichiers séquentiels.

### Pourquoi SORT est CRITIQUE ?

**Dans 30 ans de carrière mainframe, SORT est utilisé dans 80% des jobs de production.**

**Exemples réels :**
- Trier 10 millions de transactions bancaires par date/montant
- Extraire les transactions > 10,000€ pour l'audit
- Joindre un fichier clients avec un fichier transactions
- Éliminer les doublons dans un fichier master
- Reformater des données pour un système externe

**Si tu maîtrises SORT, tu es INDISPENSABLE.**

### Structure d'un Job SORT

```jcl
//stepname EXEC PGM=SORT
//SYSOUT   DD SYSOUT=*
//SORTIN   DD DSN=input-file,DISP=SHR
//SORTOUT  DD DSN=output-file,DISP=(NEW,CATLG)
//SYSIN    DD *
  SORT FIELDS=(start,length,format,order)
/*
```

**DDnames obligatoires :**
- **SYSOUT** - Messages de sortie
- **SORTIN** - Fichier d'entrée
- **SORTOUT** - Fichier de sortie
- **SYSIN** - Commandes SORT

### Tri Simple - Exemple de Base

**Trier un fichier par un champ :**

```jcl
//SORT1   JOB (ACCT),'SIMPLE SORT',CLASS=A,MSGCLASS=X
//STEP1   EXEC PGM=SORT
//SYSOUT   DD SYSOUT=*
//SORTIN   DD DSN=INPUT.DATA,DISP=SHR
//SORTOUT  DD DSN=SORTED.DATA,
//            DISP=(NEW,CATLG),
//            UNIT=SYSDA,
//            SPACE=(CYL,(10,2),RLSE)
//SYSIN    DD *
  SORT FIELDS=(1,10,CH,A)
/*
```

**Décortiquons la commande SORT :**

```
SORT FIELDS=(1,10,CH,A)
             ↑  ↑  ↑  ↑
             │  │  │  └─ A = Ascending (croissant)
             │  │  └──── CH = Character (texte)
             │  └─────── Length = 10 bytes
             └────────── Start = position 1
```

**Ça trie le fichier par les 10 premiers caractères, en ordre croissant.**

### Formats de Données SORT

**SORT supporte plusieurs formats :**

| Format | Description | Exemple |
|--------|-------------|---------|
| **CH** | Character (texte) | Noms, codes |
| **ZD** | Zoned Decimal (numérique avec signe) | Montants comptables |
| **PD** | Packed Decimal (numérique compact) | Calculs financiers |
| **BI** | Binary (entier binaire) | Compteurs |
| **FI** | Fixed-point (entier signé) | IDs numériques |
| **FL** | Floating-point (décimal) | Calculs scientifiques |

**Exemples :**

```jcl
//* Trier par nom (caractères)
SORT FIELDS=(1,20,CH,A)

//* Trier par montant (zoned decimal)
SORT FIELDS=(21,10,ZD,D)

//* Trier par ID client (binary)
SORT FIELDS=(1,4,BI,A)
```

### Ordre de Tri

**A = Ascending (Croissant)**
```
001, 002, 003, 004...
AAA, BBB, CCC, DDD...
```

**D = Descending (Décroissant)**
```
999, 998, 997, 996...
ZZZ, YYY, XXX, WWW...
```

### Tri sur Plusieurs Champs

**Tu peux trier sur plusieurs colonnes (clés) :**

```jcl
SORT FIELDS=(1,10,CH,A,11,5,ZD,D)
             ↑        ↑
             │        └─ Clé secondaire : colonnes 11-15, décroissant
             └────────── Clé primaire : colonnes 1-10, croissant
```

**Exemple réel : Transactions bancaires**

```jcl
//STEP1   EXEC PGM=SORT
//SYSOUT   DD SYSOUT=*
//SORTIN   DD DSN=BANK.TRANSACTIONS,DISP=SHR
//SORTOUT  DD DSN=BANK.SORTED.TRANS,DISP=(NEW,CATLG)
//SYSIN    DD *
  SORT FIELDS=(1,8,CH,A,     Client ID (croissant)
               9,8,CH,A,     Date (croissant)
               17,10,ZD,D)   Montant (décroissant)
/*
```

**Résultat :** Transactions triées par client, puis par date, puis par montant (du plus grand au plus petit).

### COPY - Copier Sans Trier

**Si tu veux juste copier (sans tri) mais utiliser les autres fonctions de SORT :**

```jcl
SORT FIELDS=COPY
```

**Équivalent à IEBGENER, mais SORT est plus rapide pour les gros fichiers.**

### SELECT - Filtrer les Enregistrements

**SELECT garde seulement les records qui matchent une condition.**

**Syntaxe :**
```jcl
INCLUDE COND=(start,length,format,operator,value)
```

**Opérateurs :**
- **EQ** - Equal (égal)
- **NE** - Not Equal (pas égal)
- **GT** - Greater Than (supérieur)
- **LT** - Less Than (inférieur)
- **GE** - Greater or Equal (supérieur ou égal)
- **LE** - Less or Equal (inférieur ou égal)

**Exemple : Extraire les transactions > 10000€**

```jcl
//STEP1   EXEC PGM=SORT
//SYSOUT   DD SYSOUT=*
//SORTIN   DD DSN=ALL.TRANSACTIONS,DISP=SHR
//SORTOUT  DD DSN=BIG.TRANSACTIONS,DISP=(NEW,CATLG)
//SYSIN    DD *
  SORT FIELDS=COPY
  INCLUDE COND=(21,10,ZD,GT,10000)
/*
```

**Ça copie seulement les lignes où le montant (colonnes 21-30) est > 10000.**

### OMIT - Exclure les Enregistrements

**OMIT exclut les records qui matchent une condition.**

**Exemple : Exclure les transactions annulées**

```jcl
SORT FIELDS=COPY
OMIT COND=(1,6,CH,EQ,C'CANCEL')
```

**Ça copie TOUT sauf les lignes qui commencent par "CANCEL".**

### Conditions Multiples

**AND (ET) :**

```jcl
INCLUDE COND=(1,2,CH,EQ,C'FR',AND,
              21,10,ZD,GT,5000)
```

**Garde les records où pays='FR' ET montant > 5000**

**OR (OU) :**

```jcl
INCLUDE COND=(1,2,CH,EQ,C'FR',OR,
              1,2,CH,EQ,C'BE')
```

**Garde les records où pays='FR' OU pays='BE'**

**Conditions complexes :**

```jcl
INCLUDE COND=((1,2,CH,EQ,C'FR',AND,21,10,ZD,GT,5000),OR,
              (1,2,CH,EQ,C'BE',AND,21,10,ZD,GT,3000))
```

**France avec montant > 5000 OU Belgique avec montant > 3000**

### OUTREC - Reformater la Sortie

**OUTREC te permet de réorganiser, extraire, ou modifier les champs.**

**Syntaxe :**
```jcl
OUTREC FIELDS=(field-specifications)
```

**Exemple : Extraire seulement certaines colonnes**

```jcl
//STEP1   EXEC PGM=SORT
//SYSOUT   DD SYSOUT=*
//SORTIN   DD DSN=FULL.DATA,DISP=SHR
//SORTOUT  DD DSN=EXTRACTED.DATA,DISP=(NEW,CATLG)
//SYSIN    DD *
  SORT FIELDS=COPY
  OUTREC FIELDS=(1,10,    Client ID
                 21,8,    Date
                 29,10)   Montant
/*
```

**Ça crée un fichier avec seulement ces 3 champs (28 bytes au lieu de peut-être 200).**

### Ajouter des Constantes

**Tu peux ajouter du texte fixe :**

```jcl
OUTREC FIELDS=(C'COUNTRY:',1,2,    'COUNTRY:' + code pays
               C' AMOUNT:',21,10)  ' AMOUNT:' + montant
```

**Résultat :**
```
COUNTRY:FR AMOUNT:0000125000
COUNTRY:BE AMOUNT:0000087500
```

### Conversion de Format

**EDIT - Formater les nombres**

```jcl
OUTREC FIELDS=(1,10,          Client ID
               21,10,ZD,      Montant brut
               EDIT=(IIIITTT.TT))  Formaté avec point décimal
```

**Exemple : 0000125000 devient 00001250.00**

### SUM - Agréger les Données

**SUM additionne les valeurs pour les clés identiques.**

**Exemple : Total des transactions par client**

```jcl
//STEP1   EXEC PGM=SORT
//SYSOUT   DD SYSOUT=*
//SORTIN   DD DSN=DAILY.TRANSACTIONS,DISP=SHR
//SORTOUT  DD DSN=CLIENT.TOTALS,DISP=(NEW,CATLG)
//SYSIN    DD *
  SORT FIELDS=(1,10,CH,A)      Trie par client ID
  SUM FIELDS=(21,10,ZD)        Somme les montants
/*
```

**Si tu as :**
```
CLIENT001 100
CLIENT001 200
CLIENT001 150
CLIENT002 300
```

**Tu obtiens :**
```
CLIENT001 450
CLIENT002 300
```

### Éliminer les Doublons

**Méthode 1 : SUM FIELDS=NONE**

```jcl
SORT FIELDS=(1,80,CH,A)
SUM FIELDS=NONE
```

**Garde seulement la première occurrence de chaque enregistrement unique.**

**Méthode 2 : NODUPS**

```jcl
SORT FIELDS=(1,10,CH,A),NODUPS
```

**Élimine les doublons pendant le tri.**

### SORTOF - Fichiers de Sortie Multiples

**Créer plusieurs fichiers de sortie selon des conditions.**

**Exemple : Séparer par pays**

```jcl
//STEP1   EXEC PGM=SORT
//SYSOUT   DD SYSOUT=*
//SORTIN   DD DSN=ALL.TRANSACTIONS,DISP=SHR
//SORTOF01 DD DSN=FRANCE.TRANS,DISP=(NEW,CATLG)
//SORTOF02 DD DSN=BELGIUM.TRANS,DISP=(NEW,CATLG)
//SORTOF03 DD DSN=OTHER.TRANS,DISP=(NEW,CATLG)
//SYSIN    DD *
  SORT FIELDS=COPY
  OUTFIL FNAMES=SORTOF01,INCLUDE=(1,2,CH,EQ,C'FR')
  OUTFIL FNAMES=SORTOF02,INCLUDE=(1,2,CH,EQ,C'BE')
  OUTFIL FNAMES=SORTOF03,SAVE
/*
```

**SORTOF01** = France  
**SORTOF02** = Belgique  
**SORTOF03** = Tout le reste (SAVE)

### JOIN - Joindre Deux Fichiers

**Similaire au SQL JOIN.**

**Exemple : Joindre clients et transactions**

```jcl
//STEP1   EXEC PGM=SORT
//SYSOUT   DD SYSOUT=*
//SORTIN   DD DSN=TRANSACTIONS.DATA,DISP=SHR
//SORTJNF1 DD DSN=CLIENTS.MASTER,DISP=SHR
//SORTOUT  DD DSN=JOINED.DATA,DISP=(NEW,CATLG)
//SYSIN    DD *
  JOINKEYS F1=SORTIN,FIELDS=(1,10,A)
  JOINKEYS F2=SORTJNF1,FIELDS=(1,10,A)
  REFORMAT FIELDS=(F1:1,50,F2:11,30)
  SORT FIELDS=COPY
/*
```

**Jointure sur client ID (colonnes 1-10).**

### Cas d'Usage Réels - Production Bancaire

**1. Rapport quotidien des grosses transactions**

```jcl
//BIGTRANS JOB (ACCT),'BIG TRANSACTIONS',CLASS=A,MSGCLASS=X
//STEP1   EXEC PGM=SORT
//SYSOUT   DD SYSOUT=*
//SORTIN   DD DSN=DAILY.ALL.TRANS,DISP=SHR
//SORTOUT  DD DSN=DAILY.BIG.TRANS(+1),DISP=(NEW,CATLG)
//SYSIN    DD *
  SORT FIELDS=(21,10,ZD,D)             Tri par montant décroissant
  INCLUDE COND=(21,10,ZD,GE,50000)     Seulement >= 50000€
  OUTREC FIELDS=(1,10,                 Client ID
                 11,10,                Date
                 21,10,ZD,             Montant
                 EDIT=(IIIIITTT.TT),   Formaté
                 C' EUR')              + ' EUR'
/*
```

**2. Éliminer les doublons dans un master file**

```jcl
//DEDUP   JOB (ACCT),'REMOVE DUPLICATES',CLASS=A,MSGCLASS=X
//STEP1   EXEC PGM=SORT
//SYSOUT   DD SYSOUT=*
//SORTIN   DD DSN=MASTER.WITH.DUPES,DISP=SHR
//SORTOUT  DD DSN=MASTER.CLEAN,DISP=(NEW,CATLG)
//SYSIN    DD *
  SORT FIELDS=(1,10,CH,A)              Tri par clé
  SUM FIELDS=NONE                      Élimine doublons
/*
```

**3. Statistiques mensuelles par pays**

```jcl
//STATS   JOB (ACCT),'MONTHLY STATS',CLASS=A,MSGCLASS=X
//STEP1   EXEC PGM=SORT
//SYSOUT   DD SYSOUT=*
//SORTIN   DD DSN=MONTHLY.TRANSACTIONS,DISP=SHR
//SORTOUT  DD DSN=MONTHLY.STATS,DISP=(NEW,CATLG)
//SYSIN    DD *
  SORT FIELDS=(1,2,CH,A)               Tri par pays
  SUM FIELDS=(21,10,ZD,31,5,BI)        Somme montant + nombre trans
  OUTREC FIELDS=(C'PAYS:',1,2,
                 C' TOTAL:',21,10,ZD,EDIT=(IIIITTTTTTT.TT),
                 C' COUNT:',31,5,BI)
/*
```

### Performance SORT

**SORT est ULTRA optimisé. Sur mainframe :**
- Trier 10 millions de records : **quelques secondes**
- Trier 100 millions de records : **quelques minutes**

**Tips pour optimiser :**

1. **Alloue assez d'espace de travail (SORTWORK)**

```jcl
//SORTWORK DD UNIT=SYSDA,SPACE=(CYL,(100,50))
```

2. **Utilise plusieurs SORTWORK datasets**

```jcl
//SORTWK01 DD UNIT=SYSDA,SPACE=(CYL,(50,10))
//SORTWK02 DD UNIT=SYSDA,SPACE=(CYL,(50,10))
//SORTWK03 DD UNIT=SYSDA,SPACE=(CYL,(50,10))
//SORTWK04 DD UNIT=SYSDA,SPACE=(CYL,(50,10))
```

**SORT utilisera les 4 en parallèle = BEAUCOUP plus rapide.**

3. **Utilise COPY au lieu de SORT si pas besoin de tri**

```jcl
SORT FIELDS=COPY
```

Plus rapide que de trier sur toutes les colonnes.

### Return Codes SORT

| RC | Signification |
|----|---------------|
| 0 | Succès |
| 4 | Warning (OK mais vérifier) |
| 16 | Erreur (job planté) |

### Erreurs Courantes SORT

**ERREUR #1 : "NO SPACE IN SORTWORK"**

```
ICE088I SORTWORK SPACE INSUFFICIENT
```

**Cause :** Pas assez d'espace de travail  
**Solution :** Augmente SORTWORK

---

**ERREUR #2 : "INVALID FIELD SPECIFICATION"**

```
ICE201A INVALID FIELD SPECIFICATION
```

**Cause :** Syntaxe SORT incorrecte  
**Solution :** Vérifie les positions/longueurs

---

**ERREUR #3 : "RECORD LENGTH MISMATCH"**

**Cause :** LRECL de sortie ne correspond pas  
**Solution :** Ajuste LRECL ou OUTREC

### Exercice Pratique : SORT

**Tu as un fichier de transactions avec ce format :**
```
Colonnes 1-10:   Client ID
Colonnes 11-18:  Date (YYYYMMDD)
Colonnes 19-28:  Montant (ZD)
Colonnes 29-30:  Pays (CH)
```

**Crée un job qui :**
1. Garde seulement les transactions françaises (FR)
2. Avec montant >= 1000
3. Trie par date puis montant décroissant
4. Met dans un GDG

**Solution :**

```jcl
//SORTEX  JOB (ACCT),'SORT EXERCISE',CLASS=A,MSGCLASS=X
//STEP1   EXEC PGM=SORT
//SYSOUT   DD SYSOUT=*
//SORTIN   DD DSN=ALL.TRANSACTIONS,DISP=SHR
//SORTOUT  DD DSN=FR.BIG.TRANS(+1),
//            DISP=(NEW,CATLG),
//            UNIT=SYSDA,
//            SPACE=(CYL,(10,2),RLSE)
//SORTWK01 DD UNIT=SYSDA,SPACE=(CYL,(20,10))
//SORTWK02 DD UNIT=SYSDA,SPACE=(CYL,(20,10))
//SYSIN    DD *
  SORT FIELDS=(11,8,CH,A,      Date croissant
               19,10,ZD,D)     Montant décroissant
  INCLUDE COND=((29,2,CH,EQ,C'FR'),AND,
                (19,10,ZD,GE,1000))
/*
```

---

## 14. IDCAMS - L'Outil VSAM

### Qu'est-ce qu'IDCAMS ?

**IDCAMS = Integrated Data Cluster Access Method Services**

**C'est L'OUTIL pour gérer les fichiers VSAM et bien plus.**

**IDCAMS peut :**
- Créer/supprimer des fichiers VSAM
- Copier des données (REPRO)
- Afficher des informations (LISTCAT)
- Créer des GDG bases
- Gérer les catalogues
- Exporter/importer des données
- Vérifier l'intégrité des fichiers

**Sur mainframe, IDCAMS est aussi essentiel que SORT.**

### VSAM - Introduction

**VSAM = Virtual Storage Access Method**

**VSAM permet l'accès DIRECT aux données (par clé), pas seulement séquentiel.**

**Types de VSAM :**

1. **KSDS** (Key Sequenced Data Set) - Avec index, accès par clé
2. **ESDS** (Entry Sequenced Data Set) - Séquentiel pur
3. **RRDS** (Relative Record Data Set) - Accès par numéro de record
4. **LDS** (Linear Data Set) - Données non structurées

**Le plus courant : KSDS (type base de données)**

### Structure d'un Job IDCAMS

```jcl
//stepname EXEC PGM=IDCAMS
//SYSPRINT DD SYSOUT=*
//SYSIN    DD *
  IDCAMS commands here
/*
```

**DDnames :**
- **SYSPRINT** - Messages de sortie
- **SYSIN** - Commandes IDCAMS

### DEFINE CLUSTER - Créer un VSAM KSDS

**Créer un fichier VSAM avec index :**

```jcl
//DEFVSAM JOB (ACCT),'DEFINE VSAM',CLASS=A,MSGCLASS=X
//STEP1  EXEC PGM=IDCAMS
//SYSPRINT DD SYSOUT=*
//SYSIN    DD *
  DEFINE CLUSTER (NAME(CUSTOMER.MASTER.FILE) -
                  VOLUMES(PROD01) -
                  KEYS(10 0) -
                  RECORDSIZE(200 200) -
                  TRACKS(100 20) -
                  INDEXED -
                  SHAREOPTIONS(2 3))
/*
```

**Décortiquons :**

```
NAME(CUSTOMER.MASTER.FILE)  ← Nom du dataset
VOLUMES(PROD01)             ← Volume (disque)
KEYS(10 0)                  ← Clé: 10 bytes, position 0
RECORDSIZE(200 200)         ← Longueur record: 200 bytes (average, max)
TRACKS(100 20)              ← 100 tracks primaires, 20 secondaires
INDEXED                     ← Type KSDS (avec index)
SHAREOPTIONS(2 3)           ← Options de partage (lecture/écriture)
```

### KEYS - La Clé Primaire

**Format : KEYS(length offset)**

```jcl
KEYS(10 0)    ← 10 bytes de clé, commence à la position 0
KEYS(8 20)    ← 8 bytes de clé, commence à la position 20
```

**Exemple :**

Si ton record est :
```
CUST0001JOHN DOE     NEW YORK...
↑       ↑
0       8
```

`KEYS(8 0)` = La clé est CUST0001

### SHAREOPTIONS - Contrôle d'Accès

**Format : SHAREOPTIONS(cross-region cross-system)**

**Valeurs courantes :**

| SHAREOPTIONS | Signification |
|--------------|---------------|
| (1 3) | Lecture seule, pas de cache |
| (2 3) | Lecture/écriture, contrôle minimum |
| (3 3) | Lecture/écriture, full integrity |
| (4 3) | Lecture/écriture, full integrity + recovery |

**💡 Production : Utilise (2 3) ou (3 3)**

### DELETE - Supprimer un VSAM

```jcl
//DELVSAM EXEC PGM=IDCAMS
//SYSPRINT DD SYSOUT=*
//SYSIN    DD *
  DELETE CUSTOMER.MASTER.FILE CLUSTER
/*
```

**CLUSTER supprime tout (données + index).**

### REPRO - Copier des Données

**REPRO copie des données entre datasets (VSAM ou non-VSAM).**

**Copier un VSAM vers un autre :**

```jcl
//COPYVSAM EXEC PGM=IDCAMS
//SYSPRINT DD SYSOUT=*
//SYSIN    DD *
  REPRO INFILE(INPUT) OUTFILE(OUTPUT)
/*
//INPUT  DD DSN=SOURCE.VSAM.FILE,DISP=SHR
//OUTPUT DD DSN=DEST.VSAM.FILE,DISP=OLD
```

**Copier un PS vers un VSAM :**

```jcl
//PS2VSAM EXEC PGM=IDCAMS
//SYSPRINT DD SYSOUT=*
//SYSIN    DD *
  REPRO INFILE(PSIN) OUTFILE(VSAMOUT)
/*
//PSIN    DD DSN=SEQUENTIAL.FILE,DISP=SHR
//VSAMOUT DD DSN=VSAM.FILE,DISP=OLD
```

**Copier un VSAM vers un PS (export) :**

```jcl
//VSAM2PS EXEC PGM=IDCAMS
//SYSPRINT DD SYSOUT=*
//SYSIN    DD *
  REPRO INFILE(VSAMIN) OUTFILE(PSOUT)
/*
//VSAMIN DD DSN=VSAM.FILE,DISP=SHR
//PSOUT  DD DSN=EXPORT.FILE,
//          DISP=(NEW,CATLG),
//          SPACE=(CYL,(10,2))
```

### REPRO avec Sélection

**Copier seulement certains records :**

```jcl
REPRO INFILE(INPUT) -
      OUTFILE(OUTPUT) -
      FROMKEY(CUST0100) -
      TOKEY(CUST0999)
```

**Copie seulement les clients CUST0100 à CUST0999.**

**Ou par nombre de records :**

```jcl
REPRO INFILE(INPUT) -
      OUTFILE(OUTPUT) -
      COUNT(1000)  ← Copie les 1000 premiers records
```

### LISTCAT - Afficher les Informations

**LISTCAT affiche les infos catalogue.**

**Lister un dataset spécifique :**

```jcl
//LISTCAT EXEC PGM=IDCAMS
//SYSPRINT DD SYSOUT=*
//SYSIN    DD *
  LISTCAT ENTRIES(CUSTOMER.MASTER.FILE) ALL
/*
```

**Sortie typique :**

```
CLUSTER ------- CUSTOMER.MASTER.FILE
  HISTORY
    CREATION---2025-01-20  EXPIRATION---0000-00-00
  STATISTICS
    REC-TOTAL---------1234567
    REC-DELETED-------123
    REC-INSERTED------987
    REC-UPDATED-------456
    REC-RETRIEVED-----9876543
  ALLOCATION
    SPACE-TYPE--------CYLINDER
    SPACE-PRI---------100
    SPACE-SEC---------20
  ASSOCIATIONS
    DATA----------CUSTOMER.MASTER.FILE.DATA
    INDEX---------CUSTOMER.MASTER.FILE.INDEX
```

**Lister tous les VSAM d'un HLQ :**

```jcl
LISTCAT LEVEL(PROD) VOLUME
```

### PRINT - Afficher le Contenu

**PRINT affiche les données d'un fichier.**

```jcl
//PRINTVSAM EXEC PGM=IDCAMS
//SYSPRINT DD SYSOUT=*
//SYSIN    DD *
  PRINT INFILE(VSAMIN) COUNT(100)
/*
//VSAMIN DD DSN=CUSTOMER.MASTER.FILE,DISP=SHR
```

**Affiche les 100 premiers records en hexadécimal + caractères.**

**Format CHARACTER seulement :**

```jcl
PRINT INFILE(VSAMIN) CHARACTER COUNT(10)
```

### VERIFY - Vérifier l'Intégrité

**VERIFY vérifie et répare les indicateurs de fin de fichier.**

**Utilise VERIFY après un crash ou ABEND :**

```jcl
//VERIFY EXEC PGM=IDCAMS
//SYSPRINT DD SYSOUT=*
//SYSIN    DD *
  VERIFY FILE(VSAM1)
/*
//VSAM1 DD DSN=CUSTOMER.MASTER.FILE,DISP=OLD
```

**💡 Important après un job planté qui écrivait dans un VSAM.**

### ALTER - Modifier les Attributs

**ALTER change les caractéristiques d'un VSAM.**

**Changer l'espace alloué :**

```jcl
ALTER CUSTOMER.MASTER.FILE -
      ADDVOLUMES(PROD02)
```

**Changer le nom :**

```jcl
ALTER CUSTOMER.MASTER.FILE -
      NEWNAME(CUSTOMER.MASTER.NEW)
```

### BLDINDEX - Construire un Index Alternatif

**Créer un index secondaire (Alternate Index = AIX) :**

**Étape 1 : Définir l'AIX**

```jcl
DEFINE AIX (NAME(CUSTOMER.EMAIL.INDEX) -
            RELATE(CUSTOMER.MASTER.FILE) -
            KEYS(50 150) -
            VOLUMES(PROD01) -
            TRACKS(20 5) -
            UNIQUEKEY -
            UPGRADE)
```

**Étape 2 : Construire l'AIX**

```jcl
BLDINDEX INFILE(VSAMIN) -
         OUTFILE(AIXOUT)
```

**Maintenant tu peux accéder aux customers par email en plus de l'ID !**

### DEFINE GDG - Créer une GDG Base

**On l'a déjà vu dans la Partie 2, rappel rapide :**

```jcl
//DEFGDG EXEC PGM=IDCAMS
//SYSPRINT DD SYSOUT=*
//SYSIN    DD *
  DEFINE GDG (NAME(BACKUP.DAILY.MASTER) -
              LIMIT(7) -
              SCRATCH -
              NOEMPTY)
/*
```

### EXPORT/IMPORT - Portable VSAM

**EXPORT crée une copie portable d'un VSAM (avec structure) :**

```jcl
//EXPORT EXEC PGM=IDCAMS
//SYSPRINT DD SYSOUT=*
//SYSIN    DD *
  EXPORT CUSTOMER.MASTER.FILE -
         OUTFILE(BACKUP)
/*
//BACKUP DD DSN=CUSTOMER.EXPORT.FILE,
//          DISP=(NEW,CATLG),
//          UNIT=TAPE
```

**IMPORT restaure depuis l'export :**

```jcl
//IMPORT EXEC PGM=IDCAMS
//SYSPRINT DD SYSOUT=*
//SYSIN    DD *
  IMPORT INFILE(BACKUP) -
         OUTFILE(RESTORED)
/*
//BACKUP   DD DSN=CUSTOMER.EXPORT.FILE,DISP=SHR
//RESTORED DD DSN=CUSTOMER.RESTORED.FILE,DISP=OLD
```

**💡 EXPORT/IMPORT préserve TOUTE la structure (clés, index, etc.)**

### Cas d'Usage Réels

**1. Backup quotidien VSAM vers tape :**

```jcl
//BACKUP  JOB (ACCT),'DAILY VSAM BACKUP',CLASS=A,MSGCLASS=X
//STEP1   EXEC PGM=IDCAMS
//SYSPRINT DD SYSOUT=*
//SYSIN    DD *
  REPRO INFILE(PROD) OUTFILE(BACKUP)
/*
//PROD   DD DSN=PROD.CUSTOMER.MASTER,DISP=SHR
//BACKUP DD DSN=BACKUP.CUSTOMER.DAILY(+1),
//          DISP=(NEW,CATLG),
//          UNIT=TAPE
```

**2. Reorganiser un VSAM (defrag) :**

```jcl
//REORG   JOB (ACCT),'REORG VSAM',CLASS=A,MSGCLASS=X
//*
//* Step 1: Export vers temp
//*
//STEP1   EXEC PGM=IDCAMS
//SYSPRINT DD SYSOUT=*
//SYSIN    DD *
  REPRO INFILE(OLD) OUTFILE(TEMP)
/*
//OLD  DD DSN=PROD.CUSTOMER.MASTER,DISP=SHR
//TEMP DD DSN=&&TEMPFILE,
//        DISP=(NEW,PASS),
//        SPACE=(CYL,(50,10))
//*
//* Step 2: Delete old
//*
//STEP2   EXEC PGM=IDCAMS
//SYSPRINT DD SYSOUT=*
//SYSIN    DD *
  DELETE PROD.CUSTOMER.MASTER CLUSTER
/*
//*
//* Step 3: Recreate
//*
//STEP3   EXEC PGM=IDCAMS
//SYSPRINT DD SYSOUT=*
//SYSIN    DD *
  DEFINE CLUSTER (NAME(PROD.CUSTOMER.MASTER) -
                  ... same parameters ...)
/*
//*
//* Step 4: Restore data
//*
//STEP4   EXEC PGM=IDCAMS
//SYSPRINT DD SYSOUT=*
//SYSIN    DD *
  REPRO INFILE(TEMP) OUTFILE(NEW)
/*
//TEMP DD DSN=&&TEMPFILE,DISP=(OLD,DELETE)
//NEW  DD DSN=PROD.CUSTOMER.MASTER,DISP=OLD
```

**3. Extraire des données pour analyse :**

```jcl
//EXTRACT JOB (ACCT),'EXTRACT FOR ANALYSIS',CLASS=A,MSGCLASS=X
//STEP1   EXEC PGM=IDCAMS
//SYSPRINT DD SYSOUT=*
//SYSIN    DD *
  REPRO INFILE(VSAM) -
        OUTFILE(PS) -
        FROMKEY(CUST0001) -
        TOKEY(CUST9999)
/*
//VSAM DD DSN=PROD.CUSTOMER.MASTER,DISP=SHR
//PS   DD DSN=ANALYSIS.CUSTOMER.EXTRACT,
//        DISP=(NEW,CATLG),
//        SPACE=(CYL,(10,2))
```

### Return Codes IDCAMS

| RC | Signification |
|----|---------------|
| 0 | Succès |
| 4 | Warning (OK mais vérifier) |
| 8 | Erreur (commande échouée) |
| 12 | Erreur grave |
| 16 | Erreur fatale |

### Erreurs Courantes IDCAMS

**ERREUR #1 : "NOT FOUND"**

```
IDC3009I DATASET NOT FOUND
```

**Cause :** Le dataset n'existe pas  
**Solution :** Vérifie le nom ou crée-le d'abord

---

**ERREUR #2 : "DUPLICATE KEY"**

```
IDC3351I DUPLICATE KEY
```

**Cause :** Tentative d'insérer une clé qui existe déjà  
**Solution :** Vérifie les données ou utilise UPDATE

### Exercice Pratique : IDCAMS

**Crée un job qui :**
1. Définit un VSAM KSDS pour des produits (clé = product ID sur 10 bytes)
2. Charge des données depuis un PS
3. Liste les statistiques

**Solution :**

```jcl
//VSAMEX  JOB (ACCT),'VSAM EXERCISE',CLASS=A,MSGCLASS=X
//*
//* Step 1: Define VSAM
//*
//STEP1   EXEC PGM=IDCAMS
//SYSPRINT DD SYSOUT=*
//SYSIN    DD *
  DEFINE CLUSTER (NAME(PRODUCTS.MASTER) -
                  VOLUMES(WORK01) -
                  KEYS(10 0) -
                  RECORDSIZE(150 150) -
                  TRACKS(50 10) -
                  INDEXED -
                  SHAREOPTIONS(2 3))
/*
//*
//* Step 2: Load data
//*
//STEP2   EXEC PGM=IDCAMS
//SYSPRINT DD SYSOUT=*
//SYSIN    DD *
  REPRO INFILE(PSIN) OUTFILE(VSAMOUT)
/*
//PSIN     DD DSN=PRODUCTS.SOURCE.DATA,DISP=SHR
//VSAMOUT  DD DSN=PRODUCTS.MASTER,DISP=OLD
//*
//* Step 3: List statistics
//*
//STEP3   EXEC PGM=IDCAMS
//SYSPRINT DD SYSOUT=*
//SYSIN    DD *
  LISTCAT ENTRIES(PRODUCTS.MASTER) ALL
/*
```

---

**📚 FIN DE LA PARTIE 3A (Chapitres 13-14)**

Tu as maintenant maîtrisé :
✅ SORT/DFSORT - L'utilitaire le plus puissant
✅ IDCAMS - Gestion VSAM complète

**La suite (Partie 3B) va couvrir :**
- Procedures (PROCS)
- Paramètres Symboliques  
- Conditional Processing
- JES2/JES3

**Tu veux que je continue direct ou tu veux une pause pour digérer ?** 🚀

---

**💎 100% Gratuit • Pour Tous • À Jamais**  
**🔗 GitHub : Learning Schooling Foundation**
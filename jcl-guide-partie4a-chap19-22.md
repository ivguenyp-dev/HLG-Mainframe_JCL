# JCL - Job Control Language : Guide Complet Mondial
## Partie 4A : Production Avancée (Chapitres 19-22)

**🔗 Repository GitHub :** [Learning Schooling Foundation - JCL](https://github.com/learning-schooling-foundation)

---

## Table des Matières - Partie 4A

19. [Restart et Checkpoint](#19-restart-et-checkpoint)
20. [Error Handling Avancé](#20-error-handling-avancé)
21. [Performance et Optimisation](#21-performance-et-optimisation)
22. [Sécurité et RACF](#22-sécurité-et-racf)

---

*Note : Cette Partie 4A couvre le restart, error handling, performance et sécurité. La Partie 4B couvrira le monitoring, best practices et la production réelle.*

---

## 19. Restart et Checkpoint

### Le Problème : Jobs Longs Qui Plantent

**Scénario réel :**

Tu as un batch qui tourne pendant **8 heures**. Il traite 10 millions de transactions bancaires.

**À 7h45, il plante. 💥**

**Sans restart/checkpoint :**
- 😱 Tu perds 7h45 de travail
- 😱 Tu dois relancer depuis le début
- 😱 Ça prendra encore 8 heures

**Avec restart/checkpoint :**
- ✅ Tu reprends à 7h45
- ✅ Seulement 15 minutes pour finir
- ✅ Zero perte de travail

**Le restart/checkpoint peut te sauver des JOURS de travail.**

### Qu'est-ce qu'un Checkpoint ?

**Checkpoint = Point de sauvegarde dans un job long.**

**Imagine un jeu vidéo :**
- Sans checkpoint → Game over = recommencer depuis le début
- Avec checkpoint → Game over = reprendre au dernier checkpoint

**Même concept pour les jobs mainframe.**

### Types de Restart

**1. Automatic Step Restart**
- JCL gère automatiquement
- Reprend au step qui a planté
- Pas de programmation nécessaire

**2. Checkpoint/Restart**
- Programme COBOL gère les checkpoints
- Reprend DANS le step, pas au début
- Requiert du code dans le programme

**3. Deferred Restart**
- Opérateur décide quand/où restart
- Contrôle manuel total
- Utilisé pour situations complexes

### RD Parameter - Restart Definition

**Le paramètre RD contrôle le restart automatique.**

**Syntaxe :**
```jcl
//stepname EXEC PGM=...,RD=option
```

**Options RD :**

| Option | Description |
|--------|-------------|
| **R** | Restart from this step (défaut) |
| **RNC** | Restart, No Checkpoint (ignore les checkpoints) |
| **NR** | No Restart (ne pas restart) |
| **NC** | No Checkpoint (pas de checkpoints) |

### Restart Automatique Simple

**Job initial :**

```jcl
//LONGJOB JOB (ACCT),'LONG BATCH',CLASS=A,MSGCLASS=X
//STEP1   EXEC PGM=PROG1
//DD1     DD DSN=INPUT.FILE,DISP=SHR
//STEP2   EXEC PGM=PROG2
//DD2     DD DSN=WORK.FILE,DISP=SHR
//STEP3   EXEC PGM=PROG3
//DD3     DD DSN=OUTPUT.FILE,DISP=(NEW,CATLG)
```

**STEP2 plante.**

**Pour restart depuis STEP2 :**

```jcl
//LONGJOB JOB (ACCT),'RESTART BATCH',CLASS=A,MSGCLASS=X,
//        RESTART=STEP2
//STEP1   EXEC PGM=PROG1
//DD1     DD DSN=INPUT.FILE,DISP=SHR
//STEP2   EXEC PGM=PROG2
//DD2     DD DSN=WORK.FILE,DISP=SHR
//STEP3   EXEC PGM=PROG3
//DD3     DD DSN=OUTPUT.FILE,DISP=(NEW,CATLG)
```

**RESTART=STEP2 dans la JOB card → Skip STEP1, commence à STEP2.**

### Restart avec PROC

**Si ton step utilise une PROC :**

```jcl
//STEP1 EXEC MYPROC
```

**Pour restart depuis un step DANS la PROC :**

```jcl
//        RESTART=STEP1.PROCSTEP
```

**Format : `RESTART=jobstep.procstep`**

### Checkpoint/Restart COBOL

**Pour checkpoint DANS un programme (pas juste au début d'un step).**

**Dans le programme COBOL :**

```cobol
WORKING-STORAGE SECTION.
01  CHECKPOINT-RECORD.
    05  CURRENT-RECORD-NUMBER  PIC 9(10).
    05  TOTAL-PROCESSED        PIC 9(10).
    
PROCEDURE DIVISION.
    PERFORM PROCESS-RECORDS
        VARYING RECORD-NUMBER FROM 1 BY 1
        UNTIL END-OF-FILE
        
    *> Every 10000 records, take a checkpoint
    IF RECORD-NUMBER = 10000
       MOVE RECORD-NUMBER TO CURRENT-RECORD-NUMBER
       WRITE CHECKPOINT-RECORD
       CALL 'CHKPT' USING CHECKPOINT-AREA
    END-IF.
```

**Dans le JCL :**

```jcl
//STEP1   EXEC PGM=BATCHPROG
//SYSCHK  DD DSN=CHECKPOINT.FILE,DISP=OLD
//SYSABOUT DD *
  CHKPT
/*
```

**Si le job plante :**
- Le checkpoint file contient la position
- Au restart, le programme reprend depuis ce record

### Checkpoint File (SYSCHK)

**SYSCHK = DD name standard pour checkpoint.**

```jcl
//SYSCHK DD DSN=MY.CHECKPOINT.FILE,
//          DISP=(NEW,CATLG),
//          UNIT=SYSDA,
//          SPACE=(TRK,(1,1))
```

**Ce fichier stocke :**
- Numéro du record en cours
- Compteurs
- État du programme
- Toute info nécessaire pour reprendre

### Cas Réel : Batch Bancaire Quotidien

**Job qui traite 10M de transactions :**

```jcl
//DAILYBCH JOB (ACCT),'DAILY TRANSACTIONS',CLASS=A,MSGCLASS=X
//*
//* Step 1: Extract transactions
//*
//EXTRACT EXEC PGM=EXTRACTPGM,RD=R
//INPUT    DD DSN=PROD.TRANSACTIONS.RAW,DISP=SHR
//OUTPUT   DD DSN=&&EXTRACT,DISP=(NEW,PASS),
//            SPACE=(CYL,(100,50))
//SYSCHK   DD DSN=EXTRACT.CHECKPOINT,
//            DISP=(NEW,CATLG),
//            SPACE=(TRK,(1,1))
//*
//* Step 2: Validate transactions (3 hours)
//*
//VALIDATE EXEC PGM=VALIDPGM,RD=R
//INPUT    DD DSN=&&EXTRACT,DISP=(OLD,PASS)
//OUTPUT   DD DSN=&&VALID,DISP=(NEW,PASS),
//            SPACE=(CYL,(100,50))
//REJECT   DD DSN=PROD.REJECT.FILE(+1),
//            DISP=(NEW,CATLG)
//SYSCHK   DD DSN=VALID.CHECKPOINT,
//            DISP=(NEW,CATLG),
//            SPACE=(TRK,(1,1))
//*
//* Step 3: Post to accounts (4 hours)
//*
//POST    EXEC PGM=POSTPGM,RD=R
//INPUT    DD DSN=&&VALID,DISP=(OLD,DELETE)
//ACCOUNTS DD DSN=PROD.ACCOUNTS.MASTER,DISP=OLD
//SYSCHK   DD DSN=POST.CHECKPOINT,
//            DISP=(NEW,CATLG),
//            SPACE=(TRK,(1,1))
```

**Si le job plante au step POST après 6 heures :**

```jcl
//DAILYBCH JOB (ACCT),'RESTART DAILY TRANS',CLASS=A,MSGCLASS=X,
//         RESTART=POST
//... (même JCL qu'avant)
```

**Résultat :**
- Skip EXTRACT (déjà fait)
- Skip VALIDATE (déjà fait)
- Restart POST depuis le dernier checkpoint
- Seulement 30 minutes au lieu de 8 heures !

### Automatic Restart Facility (ARF)

**z/OS peut restart automatiquement un job qui ABEND.**

**Setup dans JES2 :**

```jcl
/*JOBPARM RESTART=Y
```

**Ou dans la JOB card :**

```jcl
//        RESTART=*
```

**`RESTART=*` = Restart automatiquement au step qui a planté.**

### Considérations Importantes

**1. Datasets DISP**

Attention aux DISP si tu restart :

```jcl
// Mauvais pour restart :
//OUTPUT DD DSN=FINAL.FILE,DISP=(NEW,CATLG)
// → Si restart, le fichier existe déjà !

// Bon pour restart :
//OUTPUT DD DSN=FINAL.FILE,DISP=(MOD,CATLG)
// → Append si existe
```

**2. GDG avec Restart**

```jcl
//OUTPUT DD DSN=BACKUP.DAILY(+1),DISP=(NEW,CATLG)
```

**Au restart, `(+1)` crée une NOUVELLE génération !**

**Solution : Utilise le nom absolu au restart**

```jcl
//OUTPUT DD DSN=BACKUP.DAILY.G0001V00,DISP=OLD
```

**3. Temp Datasets (&&)**

Les `&&` datasets sont supprimés entre deux runs.

**Solution : Utilise DISP=(NEW,PASS) et nomme-les**

```jcl
//TEMP DD DSN=WORK.TEMPFILE,DISP=(NEW,PASS)
```

### Erreurs Courantes

**ERREUR #1 : RESTART step inexistant**

```
IEF877I JOB FAILED - RESTART STEP NOT FOUND
```

**Cause :** Le stepname dans RESTART= n'existe pas  
**Solution :** Vérifie le nom du step

---

**ERREUR #2 : Dataset already exists**

```
IEF703I DATASET ALREADY CATALOGED
```

**Cause :** DISP=(NEW,CATLG) mais fichier existe déjà  
**Solution :** Change en DISP=(MOD,CATLG) ou delete le fichier

---

**ERREUR #3 : Checkpoint file not found**

```
UNABLE TO OPEN CHECKPOINT FILE
```

**Cause :** SYSCHK DD manquant ou fichier supprimé  
**Solution :** Crée le fichier ou restart sans checkpoint

### Best Practices Restart

**1. Utilise toujours RD=R pour jobs longs**

```jcl
//STEP1 EXEC PGM=LONGPROG,RD=R
```

**2. Prends des checkpoints réguliers**

```cobol
*> Every 10000 records
IF MOD(RECORD-COUNT, 10000) = 0
   PERFORM TAKE-CHECKPOINT
END-IF
```

**3. Documente les points de restart**

```jcl
//*****************************************
//* RESTART INSTRUCTIONS:
//* If ABEND in STEP2, restart with:
//*   //   RESTART=STEP2
//* If ABEND in STEP3, ensure WORK.FILE
//* exists before restart
//*****************************************
```

**4. Teste le restart en DEV**

Ne découvre pas que ton restart ne marche pas en PROD à 3h du matin !

### Exercice Pratique : Restart

**Crée un job avec 3 steps. Simule un ABEND au step 2. Restart depuis le step 2.**

**Solution :**

**Job initial :**

```jcl
//TESTREST JOB (ACCT),'TEST RESTART',CLASS=A,MSGCLASS=X
//STEP1   EXEC PGM=IEFBR14
//DD1     DD DSN=WORK.STEP1.OUTPUT,
//           DISP=(NEW,CATLG),
//           SPACE=(TRK,(1,1))
//STEP2   EXEC PGM=ABENDPGM  ← Ce pgm va ABEND
//DD2     DD DSN=WORK.STEP2.OUTPUT,
//           DISP=(NEW,CATLG),
//           SPACE=(TRK,(1,1))
//STEP3   EXEC PGM=IEFBR14
//DD3     DD DSN=WORK.STEP3.OUTPUT,
//           DISP=(NEW,CATLG),
//           SPACE=(TRK,(1,1))
```

**Job plante à STEP2. Pour restart :**

```jcl
//TESTREST JOB (ACCT),'RESTART FROM STEP2',CLASS=A,MSGCLASS=X,
//         RESTART=STEP2
//STEP1   EXEC PGM=IEFBR14
//DD1     DD DSN=WORK.STEP1.OUTPUT,DISP=OLD  ← Change en OLD
//STEP2   EXEC PGM=GOODPGM  ← Remplace par un bon pgm
//DD2     DD DSN=WORK.STEP2.OUTPUT,
//           DISP=(NEW,CATLG),
//           SPACE=(TRK,(1,1))
//STEP3   EXEC PGM=IEFBR14
//DD3     DD DSN=WORK.STEP3.OUTPUT,
//           DISP=(NEW,CATLG),
//           SPACE=(TRK,(1,1))
```

---

## 20. Error Handling Avancé

### Pourquoi l'Error Handling est Critique ?

**En production, les erreurs VONT arriver :**
- Fichiers manquants
- Données corrompues
- Manque d'espace disque
- Timeouts réseau
- ABEND programmes
- Contraintes VSAM

**Sans error handling :**
- 😱 Le job plante silencieusement
- 😱 Tu découvres l'erreur le lendemain
- 😱 Les données sont incohérentes
- 😱 Personne n'est notifié

**Avec bon error handling :**
- ✅ Detection immédiate
- ✅ Notification automatique
- ✅ Rollback si nécessaire
- ✅ Logs complets

### Return Code Strategy

**Définis une stratégie claire pour les RC :**

| RC Range | Signification | Action |
|----------|---------------|--------|
| **0** | Succès parfait | Continue normalement |
| **1-4** | Warning acceptable | Continue, log |
| **5-8** | Warning sérieux | Continue, alerte |
| **9-12** | Erreur | Stop, rollback, alerte |
| **13+** | Erreur fatale | Stop immédiat, alerte urgente |

### Tester les Return Codes

**Méthode moderne avec IF/THEN :**

```jcl
//STEP1 EXEC PGM=PROCESS
//IF1 IF (STEP1.RC = 0) THEN
//  SUCCESS EXEC PGM=CONTINUE
//ELSE
//  IF (STEP1.RC <= 4) THEN
//    WARNING EXEC PGM=LOGWARNING
//    CONTINUE EXEC PGM=CONTINUE
//  ELSE
//    ERROR EXEC PGM=ROLLBACK
//    ALERT EXEC PGM=SENDALERT
//  ENDIF
//ENDIF
```

### MAXCC - Maximum Condition Code

**MAXCC = Le pire RC de tous les steps précédents.**

```jcl
//STEP1 EXEC PGM=PROG1  ← RC=0
//STEP2 EXEC PGM=PROG2  ← RC=4
//STEP3 EXEC PGM=PROG3  ← RC=0
//IF1 IF (MAXCC > 0) THEN  ← MAXCC=4 (le max)
//ALERT EXEC PGM=NOTIFY
//ENDIF
```

**Utile pour vérifier si QUELQUE CHOSE a foiré, même si c'était juste un warning.**

### SET Statement pour RC

**Tu peux forcer un RC avec SET :**

```jcl
//STEP1 EXEC PGM=PROGRAM
//IF1 IF (STEP1.RC > 8) THEN
//  SET MAXCC=16  ← Force le job à RC=16
//  ALERT EXEC PGM=SENDALERT
//ENDIF
```

**Utile pour escalader un RC.**

### ABEND Handling

**ABEND = Abnormal END (crash du programme)**

**Types d'ABEND :**

**System ABEND (Sxxx) :**
- S0C4 - Protection exception (mauvais pointeur)
- S0C7 - Data exception (donnée non-numérique)
- S0CB - Division par zéro
- S806 - Programme non trouvé
- S822 - Region trop petite
- SB37 - Espace disque plein

**User ABEND (Uxxx) :**
- U0001-U4095 - Définis par le programmeur

### Détecter les ABEND

```jcl
//STEP1 EXEC PGM=PROGRAM
//IF1 IF (STEP1.ABENDED) THEN
//  CLEANUP EXEC PGM=ROLLBACK
//  ALERT EXEC PGM=SENDALERT,PARM='STEP1 ABENDED'
//ENDIF
```

### Tester un ABEND Spécifique

```jcl
//IF1 IF (STEP1.ABENDCC = 'S0C7') THEN
//  ALERT EXEC PGM=NOTIFY,PARM='DATA ERROR STEP1'
//ENDIF
```

### Ignore ABEND (COND=EVEN)

**Pour exécuter un step même si ABEND :**

```jcl
//STEP1   EXEC PGM=MAINPROG
//CLEANUP EXEC PGM=CLEANUP,COND=EVEN
```

**CLEANUP tourne même si STEP1 ABEND.**

### Rollback Pattern

**Pattern standard pour transaction rollback :**

```jcl
//TRANSACT JOB (ACCT),'TRANSACTION WITH ROLLBACK',
//         CLASS=A,MSGCLASS=X
//*
//* Step 1: Backup before processing
//*
//BACKUP  EXEC PGM=IDCAMS
//SYSIN    DD *
  REPRO INFILE(PROD) OUTFILE(BKP)
/*
//PROD DD DSN=PROD.MASTER.FILE,DISP=SHR
//BKP  DD DSN=BACKUP.MASTER.FILE,DISP=(NEW,CATLG)
//*
//* Step 2: Process transactions
//*
//PROCESS EXEC PGM=TRANSACTIONPGM
//MASTER  DD DSN=PROD.MASTER.FILE,DISP=OLD
//TRANS   DD DSN=DAILY.TRANSACTIONS,DISP=SHR
//*
//* Step 3: If process OK, commit
//*
//IF1 IF (PROCESS.RC = 0) THEN
//COMMIT  EXEC PGM=COMMITPGM
//ENDIF
//*
//* Step 4: If process failed, rollback
//*
//IF2 IF (PROCESS.RC > 0 OR PROCESS.ABENDED) THEN
//ROLLBACK EXEC PGM=IDCAMS
//SYSIN    DD *
  DELETE PROD.MASTER.FILE CLUSTER
  REPRO INFILE(BKP) OUTFILE(PROD)
/*
//BKP  DD DSN=BACKUP.MASTER.FILE,DISP=SHR
//PROD DD DSN=PROD.MASTER.FILE,DISP=OLD
//ALERT EXEC PGM=SENDALERT,PARM='ROLLBACK EXECUTED'
//ENDIF
```

### Notification Pattern

**Envoyer des alerts selon le résultat :**

```jcl
//BATCH JOB (ACCT),'BATCH WITH NOTIFICATION',CLASS=A
//PROCESS EXEC PGM=BATCHPGM
//IF1 IF (PROCESS.RC = 0) THEN
//  NOTIFY EXEC PGM=SENDMAIL,
//         PARM='TO=ops@bank.com SUBJECT=Batch Success'
//ELSE
//  IF (PROCESS.RC <= 8) THEN
//    NOTIFY EXEC PGM=SENDMAIL,
//           PARM='TO=ops@bank.com SUBJECT=Batch Warning'
//  ELSE
//    NOTIFY EXEC PGM=SENDMAIL,
//           PARM='TO=ops@bank.com,manager@bank.com',
//           PARM='SUBJECT=Batch FAILED - Urgent'
//    PAGER EXEC PGM=SENDPAGE,PARM='555-1234'
//  ENDIF
//ENDIF
```

### Error Logging

**Log tous les problèmes pour debug :**

```jcl
//STEP1   EXEC PGM=MAINPROG
//SYSPRINT DD SYSOUT=*
//ERRORLOG DD DSN=ERROR.LOG.FILE(+1),
//            DISP=(NEW,CATLG),
//            DCB=(RECFM=VB,LRECL=200)
//IF1 IF (STEP1.RC > 0 OR STEP1.ABENDED) THEN
//LOG EXEC PGM=LOGGER
//SYSIN DD *
JOB: &JOBNAME
STEP: STEP1
RC: &STEP1.RC
ABEND: &STEP1.ABENDCC
TIME: &LTIME
DATE: &LDATE
/*
//LOGFILE DD DSN=ERROR.LOG.FILE(0),DISP=MOD
//ENDIF
```

### Retry Logic

**Retry un step X fois avant d'abandonner :**

```jcl
//RETRY1  EXEC PGM=NETWORKCALL
//IF1 IF (RETRY1.RC > 0) THEN
//  WAIT EXEC PGM=SLEEP,PARM='30'  ← Attends 30 sec
//  RETRY2 EXEC PGM=NETWORKCALL
//  IF2 IF (RETRY2.RC > 0) THEN
//    WAIT2 EXEC PGM=SLEEP,PARM='60'  ← Attends 1 min
//    RETRY3 EXEC PGM=NETWORKCALL
//    IF3 IF (RETRY3.RC > 0) THEN
//      ALERT EXEC PGM=SENDALERT,PARM='3 RETRIES FAILED'
//    ENDIF
//  ENDIF
//ENDIF
```

### Dependency Check Pattern

**Vérifier que les prérequis sont OK avant de commencer :**

```jcl
//DEPCHECK JOB (ACCT),'CHECK DEPENDENCIES',CLASS=A
//*
//* Step 1: Check if input file exists
//*
//CHECK1  EXEC PGM=IDCAMS
//SYSIN    DD *
  LISTCAT ENTRIES(INPUT.FILE.DAILY)
/*
//*
//* Step 2: Check if master file is available
//*
//CHECK2  EXEC PGM=TESTLOCK
//MASTER   DD DSN=PROD.MASTER.FILE,DISP=SHR
//*
//* Step 3: Only process if all checks passed
//*
//IF1 IF (CHECK1.RC = 0 AND CHECK2.RC = 0) THEN
//PROCESS EXEC PGM=MAINPROCESS
//ELSE
//  ALERT EXEC PGM=SENDALERT,
//        PARM='DEPENDENCIES NOT MET'
//  SET MAXCC=16
//ENDIF
```

### Best Practices Error Handling

**1. Toujours logger les erreurs**

```jcl
//ERRORLOG DD DSN=ERROR.LOG(+1),DISP=(NEW,CATLG)
```

**2. Notifier les bonnes personnes**

```
RC 0-4   → Log seulement
RC 5-8   → Email ops
RC 9-12  → Email ops + manager
RC 13+   → Email ops + manager + page on-call
ABEND    → Email ops + manager + page on-call + SMS
```

**3. Avoir un plan de rollback**

Toujours backup avant modification critique.

**4. Documenter les codes d'erreur**

```jcl
//* RC CODES:
//*  0  = Success
//*  4  = Warning - records skipped
//*  8  = Error - partial failure
//* 12  = Error - no records processed
//* 16  = Fatal - file not found
```

**5. Tester les error paths**

Ne découvre pas que ton error handling ne marche pas en prod !

---

## 21. Performance et Optimisation

### Pourquoi l'Optimisation est Critique ?

**Exemple réel bancaire :**

**Job non-optimisé :**
- Tri de 10M transactions → **45 minutes**
- Backup VSAM → **2 heures**
- Batch de nuit → **6 heures** (rate la fenêtre de 4h)

**Job optimisé :**
- Tri de 10M transactions → **8 minutes** 🔥
- Backup VSAM → **25 minutes** 🔥
- Batch de nuit → **1h30** 🔥 (dans la fenêtre !)

**Performance = La différence entre un job qui marche et un job qui coûte des millions.**

### REGION Parameter - Mémoire Allouée

**REGION = Quantité de mémoire pour le step.**

**Syntaxe :**
```jcl
//stepname EXEC PGM=...,REGION=size
```

**Unités :**
- **K** - Kilobytes
- **M** - Megabytes
- **G** - Gigabytes (modern systems)

**Exemples :**

```jcl
// Petit programme
//STEP1 EXEC PGM=SMALLPGM,REGION=4M

// SORT normal
//STEP2 EXEC PGM=SORT,REGION=64M

// Gros batch
//STEP3 EXEC PGM=BIGBATCH,REGION=256M

// TRÈS gros (moderne)
//STEP4 EXEC PGM=HUGEBATCH,REGION=2G
```

### Pourquoi REGION est Important ?

**REGION trop petite :**
- ABEND S822 (insufficient storage)
- Programme plante
- Performance dégradée (swapping)

**REGION trop grande :**
- Gaspille la mémoire
- Limite le nombre de jobs concurrents
- Pas de bénéfice réel

**Optimal :**
- Assez pour le programme + buffers
- Pas plus que nécessaire

### REGION pour SORT

**SORT bénéficie ÉNORMÉMENT de REGION.**

**Impact réel :**

```jcl
// REGION petit = SLOW
//SORT1 EXEC PGM=SORT,REGION=4M
// → 45 minutes pour 10M records

// REGION moyen = BETTER
//SORT2 EXEC PGM=SORT,REGION=32M
// → 15 minutes pour 10M records

// REGION gros = FAST
//SORT3 EXEC PGM=SORT,REGION=128M
// → 8 minutes pour 10M records
```

**Règle de base pour SORT :**
```
REGION = (taille fichier / 10) + 32M minimum
```

**Pour 100MB file → REGION=40M**  
**Pour 1GB file → REGION=132M**

### BUFNO - Buffer Number

**BUFNO = Nombre de buffers I/O pour un dataset.**

**Plus de buffers = Moins d'I/O physiques = Plus rapide.**

**Syntaxe :**
```jcl
//DD DSN=...,DISP=...,BUFNO=n
```

**Défaut :** 5 buffers

**Optimal :** 20-50 pour gros fichiers

**Exemple :**

```jcl
// Sans BUFNO (défaut = 5)
//INPUT DD DSN=BIG.FILE,DISP=SHR
// → Beaucoup d'I/O, lent

// Avec BUFNO optimisé
//INPUT DD DSN=BIG.FILE,DISP=SHR,BUFNO=30
// → Moins d'I/O, plus rapide
```

### BLKSIZE - Block Size

**BLKSIZE = Taille d'un bloc physique sur disque.**

**Plus gros bloc = Moins d'I/O = Plus rapide.**

**Optimal moderne :**
```jcl
//DD DSN=...,
//   SPACE=...,
//   DCB=(RECFM=FB,LRECL=80,BLKSIZE=27920)
```

**27920 est un "half-track" sur 3390 DASD (optimal).**

### SORT Work Files

**SORT utilise des work files pour tri intermédiaire.**

**Plus de work files = Tri parallélisé = Plus rapide.**

**Standard :**

```jcl
//SORT    EXEC PGM=SORT,REGION=128M
//SORTWK01 DD UNIT=SYSDA,SPACE=(CYL,(100))
//SORTWK02 DD UNIT=SYSDA,SPACE=(CYL,(100))
//SORTWK03 DD UNIT=SYSDA,SPACE=(CYL,(100))
//SORTWK04 DD UNIT=SYSDA,SPACE=(CYL,(100))
```

**Optimisé (plus de work files) :**

```jcl
//SORT    EXEC PGM=SORT,REGION=256M
//SORTWK01 DD UNIT=SYSDA,SPACE=(CYL,(200))
//SORTWK02 DD UNIT=SYSDA,SPACE=(CYL,(200))
//SORTWK03 DD UNIT=SYSDA,SPACE=(CYL,(200))
//SORTWK04 DD UNIT=SYSDA,SPACE=(CYL,(200))
//SORTWK05 DD UNIT=SYSDA,SPACE=(CYL,(200))
//SORTWK06 DD UNIT=SYSDA,SPACE=(CYL,(200))
//SORTWK07 DD UNIT=SYSDA,SPACE=(CYL,(200))
//SORTWK08 DD UNIT=SYSDA,SPACE=(CYL,(200))
```

**Impact :**
- 4 work files → 15 min
- 8 work files → 8 min

### VIO - Virtual I/O

**VIO = I/O dans la mémoire au lieu du disque.**

**Pour datasets temporaires petits :**

```jcl
//TEMP DD DSN=&&TEMP,
//        DISP=(NEW,PASS),
//        UNIT=VIO,  ← En mémoire !
//        SPACE=(TRK,(10,5))
```

**Avantages :**
- ✅ TRÈS rapide (RAM speed)
- ✅ Pas d'I/O physique
- ✅ Réduit la charge DASD

**Limites :**
- ❌ Seulement pour petits fichiers (<1000 tracks)
- ❌ Perdu si job crash

**Utilise VIO pour :**
- Temp files entre steps
- Fichiers intermédiaires
- Work files de taille modeste

### Cas Réel : SORT Optimisé vs Non-Optimisé

**AVANT (non-optimisé) :**

```jcl
//SLOWSORT JOB (ACCT),'SLOW SORT',CLASS=A
//SORT    EXEC PGM=SORT
//SYSOUT   DD SYSOUT=*
//SORTIN   DD DSN=INPUT.10M.RECORDS,DISP=SHR
//SORTOUT  DD DSN=OUTPUT.SORTED,
//            DISP=(NEW,CATLG),
//            SPACE=(CYL,(100,50))
//SYSIN    DD *
  SORT FIELDS=(1,10,CH,A)
/*
// Runtime: 45 minutes 😱
```

**APRÈS (optimisé) :**

```jcl
//FASTSORT JOB (ACCT),'FAST SORT',CLASS=A
//SORT    EXEC PGM=SORT,REGION=256M  ← Big REGION
//SYSOUT   DD SYSOUT=*
//SORTIN   DD DSN=INPUT.10M.RECORDS,
//            DISP=SHR,
//            BUFNO=50  ← More buffers
//SORTOUT  DD DSN=OUTPUT.SORTED,
//            DISP=(NEW,CATLG),
//            SPACE=(CYL,(100,50)),
//            BUFNO=50,
//            DCB=BLKSIZE=27920  ← Optimal block
//SORTWK01 DD UNIT=SYSDA,SPACE=(CYL,(200))
//SORTWK02 DD UNIT=SYSDA,SPACE=(CYL,(200))
//SORTWK03 DD UNIT=SYSDA,SPACE=(CYL,(200))
//SORTWK04 DD UNIT=SYSDA,SPACE=(CYL,(200))
//SORTWK05 DD UNIT=SYSDA,SPACE=(CYL,(200))
//SORTWK06 DD UNIT=SYSDA,SPACE=(CYL,(200))
//SORTWK07 DD UNIT=SYSDA,SPACE=(CYL,(200))
//SORTWK08 DD UNIT=SYSDA,SPACE=(CYL,(200))
//SYSIN    DD *
  SORT FIELDS=(1,10,CH,A)
  OPTION DYNALLOC=(SYSDA,8),SIZE=E10000000
/*
// Runtime: 8 minutes ! 🔥
```

**Optimisations appliquées :**
1. REGION=256M (au lieu de défaut ~4M)
2. BUFNO=50 pour input/output
3. BLKSIZE optimal (27920)
4. 8 work files (au lieu de 0)
5. DYNALLOC pour allocation dynamique
6. SIZE estimation pour meilleur algorithme

**Résultat : 5.6x plus rapide !**

### Best Practices Performance

**1. REGION généreux pour SORT**

```jcl
//SORT EXEC PGM=SORT,REGION=256M
```

**2. BUFNO pour gros fichiers**

```jcl
//DD DSN=BIG.FILE,DISP=SHR,BUFNO=30-50
```

**3. VIO pour temp files**

```jcl
//TEMP DD UNIT=VIO,SPACE=(TRK,(10))
```

**4. BLKSIZE optimal**

```jcl
DCB=BLKSIZE=27920  ← Ou laisse system-determined
```

**5. Plusieurs work files pour SORT**

```jcl
//SORTWK01-08 DD ...  ← 8 work files
```

---

## 22. Sécurité et RACF

### Qu'est-ce que RACF ?

**RACF = Resource Access Control Facility**

**RACF est le système de sécurité d'IBM mainframe.**

**RACF contrôle :**
- Qui peut accéder à quoi
- Quand
- Comment

**Analogie :**

RACF est comme un **bouncer de boîte de nuit** :
- Vérifie ton ID (userid + password)
- Check la liste VIP (authorizations)
- Te laisse entrer ou te refuse

**Sans RACF :** N'importe qui peut accéder aux salaires, numéros de carte bancaire, données médicales. 😱

**Avec RACF :** Access strictly controlled. ✅

### Pourquoi RACF est CRITIQUE en Banque ?

**Les données mainframe sont ULTRA-SENSIBLES :**
- 💰 Comptes bancaires
- 💳 Numéros de carte
- 🔐 Transactions financières
- 📊 Données clients
- 💼 Salaires employés

**Un breach = Catastrophe :**
- Millions d'€ de perte
- Amendes réglementaires massives
- Réputation détruite
- Risque pénal

**RACF = La dernière ligne de défense.**

### Concepts RACF de Base

**1. User ID**

Ton identifiant unique sur le mainframe.

```
Exemple: USER001, JOHNDOE, PAYROLL
```

**2. Group**

Collection d'utilisateurs.

```
Exemple: DEVELOPERS, OPERATORS, MANAGERS
```

**3. Resource**

Ce qui est protégé (dataset, programme, transaction).

```
Exemple: PROD.CUSTOMER.MASTER, PAYROLL.EXE
```

**4. Profile**

Règles d'accès pour un resource.

```
Profile: PROD.CUSTOMER.MASTER
  - USER001: READ
  - USER002: UPDATE
  - GROUP MANAGERS: READ
```

**5. Authority Level**

Niveau de permission.

| Level | Permission |
|-------|------------|
| **NONE** | Aucun accès |
| **READ** | Lecture seulement |
| **UPDATE** | Lecture + Modification |
| **CONTROL** | Tout + Grant access |
| **ALTER** | Tout + Changer le profile |

### RACF et JCL

**RACF vérifie chaque DD statement de ton JCL.**

```jcl
//STEP1 EXEC PGM=PROG1
//INPUT DD DSN=PROD.CUSTOMER.DATA,DISP=SHR
       ↑
       RACF check: USER001 peut lire ce dataset ?
```

**Si RACF refuse :**
```
ICH408I USER(USER001) GROUP(DEVS) 
  DSN(PROD.CUSTOMER.DATA) 
  VOL(PROD01) INSUFFICIENT ACCESS AUTHORITY
```

**Le job plante avec RACF violation.**

### Protéger un Dataset avec RACF

**Commande RACF (faite par Security Admin) :**

```racf
ADDSD 'PROD.CUSTOMER.MASTER' UACC(NONE)
```

**UACC = Universal Access Authority (défaut pour tout le monde)**

**NONE = Personne n'a accès par défaut.**

**Ensuite, grant access spécifique :**

```racf
PERMIT 'PROD.CUSTOMER.MASTER' ID(USER001) ACCESS(READ)
PERMIT 'PROD.CUSTOMER.MASTER' ID(USER002) ACCESS(UPDATE)
PERMIT 'PROD.CUSTOMER.MASTER' ID(MANAGERS) ACCESS(READ)
```

**Maintenant :**
- USER001 peut lire
- USER002 peut lire + modifier
- Groupe MANAGERS peut lire
- Tout le monde d'autre : DENIED

### Generic Profiles

**Protéger plusieurs datasets avec un pattern.**

```racf
ADDSD 'PROD.**' UACC(NONE)
```

**`**` = Wildcard (n'importe quoi)**

**Ça protège :**
- PROD.CUSTOMER.MASTER
- PROD.TRANSACTIONS.DAILY
- PROD.ACCOUNTS.HISTORY
- PROD.n'importe quoi

### Audit Logging

**RACF peut logger TOUS les accès.**

```racf
ALTDSD 'PROD.CUSTOMER.MASTER' AUDIT(ALL(READ))
```

**Maintenant, chaque READ est loggé :**

```
USER001 accessed PROD.CUSTOMER.MASTER at 2025-01-20 14:35:22
USER002 accessed PROD.CUSTOMER.MASTER at 2025-01-20 14:37:18
USER003 DENIED access to PROD.CUSTOMER.MASTER at 2025-01-20 14:38:45
```

**En banque, c'est OBLIGATOIRE pour les données sensibles.**

### Best Practices RACF

**1. Principe du moindre privilège**

Donne seulement l'accès minimum nécessaire.

```
READ si lecture suffit (pas UPDATE)
```

**2. Utilise des groupes, pas des users individuels**

```racf
/* Mauvais */
PERMIT 'PROD.DATA' ID(USER001) ACCESS(READ)
PERMIT 'PROD.DATA' ID(USER002) ACCESS(READ)
PERMIT 'PROD.DATA' ID(USER003) ACCESS(READ)

/* Bon */
PERMIT 'PROD.DATA' ID(DEVELOPERS) ACCESS(READ)
```

**3. Audit les datasets sensibles**

```racf
AUDIT(ALL(READ))  ← Log tous les accès
```

**4. Sépare PROD/TEST/DEV**

```
PROD.* → Ultra-protégé
TEST.* → Protégé
DEV.*  → Plus libre
```

**5. Jamais de passwords dans le JCL**

```jcl
/* Mauvais */
//DD DSN=SECRET.DATA,PASSWORD=PASS123

/* Bon */
//DD DSN=SECRET.DATA,DISP=SHR  ← RACF gère
```

---

**📚 FIN DE LA PARTIE 4A (Chapitres 19-22)**

Tu as maintenant maîtrisé :
✅ **Restart et Checkpoint** - Reprendre après erreur
✅ **Error Handling Avancé** - Gérer les problèmes
✅ **Performance et Optimisation** - Rendre les jobs rapides
✅ **Sécurité et RACF** - Protéger les données

**La suite (Partie 4B) va couvrir :**
- Monitoring et Diagnostics
- Best Practices Production
- JCL en Production Réelle

**Continue avec la Partie 4B !** 🚀

---

**💎 100% Gratuit • Pour Tous • À Jamais**  
**🔗 GitHub : Learning Schooling Foundation**

---

## Pour Qui On Fait Ça ?

**Pour le dev de 22 ans à Kinshasa.**  
**Pour la mère célibataire à São Paulo qui se reconvertit.**  
**Pour l'étudiant tunisien sans accès aux certifs payantes.**  
**Pour tous ceux que le système exclut par le prix.**

**Le savoir tech élite ne devrait PAS coûter $2000.**  
**Il devrait être gratuit. Pour toujours.**

**C'est notre mission. 💚**
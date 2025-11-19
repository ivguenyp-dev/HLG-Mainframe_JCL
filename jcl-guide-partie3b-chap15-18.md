# JCL - Job Control Language : Guide Complet Mondial
## Partie 3B : Automation et Job Control (Chapitres 15-18)

**🔗 Repository GitHub :** [Learning Schooling Foundation - JCL](https://github.com/learning-schooling-foundation)

---

## Table des Matières - Partie 3B

15. [Procedures (PROCS)](#15-procedures-procs)
16. [Paramètres Symboliques](#16-paramètres-symboliques)
17. [Conditional Processing](#17-conditional-processing)
18. [JES2/JES3 - Job Entry Subsystem](#18-jes2jes3---job-entry-subsystem)

---

*Note : Cette Partie 3B couvre l'automatisation et le contrôle avancé des jobs. La Partie 4 couvrira la production, l'optimisation et les best practices.*

---

## 15. Procedures (PROCS)

### Qu'est-ce qu'une PROC ?

**PROC = PROCedure = Un morceau de JCL réutilisable.**

**Le problème sans PROC :**

Tu as 100 programmes COBOL à compiler. Sans PROC, tu copies-colles le même JCL 100 fois :

```jcl
//COMPILE1 JOB ...
//STEP1 EXEC PGM=IGYCRCTL
//STEPLIB DD DSN=IGY.SIGYCOMP,DISP=SHR
//SYSPRINT DD SYSOUT=*
//SYSLIN DD DSN=&&LOADSET,DISP=(MOD,PASS)
//SYSIN DD DSN=SOURCE.COBOL(PROG1),DISP=SHR
//...

//COMPILE2 JOB ...
//STEP1 EXEC PGM=IGYCRCTL
//STEPLIB DD DSN=IGY.SIGYCOMP,DISP=SHR
//SYSPRINT DD SYSOUT=*
//SYSLIN DD DSN=&&LOADSET,DISP=(MOD,PASS)
//SYSIN DD DSN=SOURCE.COBOL(PROG2),DISP=SHR
//...

// 98 autres fois... 😱
```

**Le problème ?**
- 🚫 Copier-coller = erreurs
- 🚫 Changer quelque chose = modifier 100 jobs
- 🚫 Maintenance impossible

**La solution avec PROC :**

Tu crées UNE FOIS la procédure de compilation, tu l'utilises 100 fois.

```jcl
//COMPILE1 JOB ...
//STEP1 EXEC COBCOMP  ← Appelle la PROC

//COMPILE2 JOB ...
//STEP1 EXEC COBCOMP  ← Même PROC

//COMPILE3 JOB ...
//STEP1 EXEC COBCOMP  ← Même PROC
```

**1 modification dans la PROC = 100 jobs mis à jour automatiquement.**

### Pourquoi les PROCS sont ESSENTIELLES ?

**Dans 30 ans de carrière mainframe, tu utiliseras des PROCS dans 90% des jobs.**

**Cas d'usage réels :**
- Compiler du COBOL (même steps à chaque fois)
- Link-edit des programmes (toujours pareil)
- Backup de datasets (même structure)
- Lancer des batch standards (même pattern)

**Si tu maîtrises les PROCS, tu multiplies ta productivité par 10.**

### Anatomie d'une PROC

**Une PROC ressemble à du JCL, mais sans la JOB card.**

```jcl
//PROCNAME PROC
//STEP1    EXEC PGM=...
//DD1      DD DSN=...,DISP=...
//DD2      DD DSN=...,DISP=...
//STEP2    EXEC PGM=...
//DD3      DD DSN=...,DISP=...
//         PEND
```

**Différences avec un JOB :**
- ✅ Commence par `//PROCNAME PROC`
- ✅ Finit par `// PEND`
- ❌ Pas de JOB card
- ❌ Pas de paramètres JOB (CLASS, MSGCLASS, etc.)

### Créer une PROC Simple

**Exemple : PROC pour copier un fichier**

```jcl
//COPYFILE PROC
//COPY     EXEC PGM=IEBGENER
//SYSPRINT DD SYSOUT=*
//SYSUT1   DD DSN=INPUT.FILE,DISP=SHR
//SYSUT2   DD DSN=OUTPUT.FILE,
//            DISP=(NEW,CATLG),
//            UNIT=SYSDA,
//            SPACE=(CYL,(5,1))
//SYSIN    DD DUMMY
//         PEND
```

**Comment l'utiliser :**

```jcl
//COPYJOB JOB (ACCT),'COPY FILE',CLASS=A,MSGCLASS=X
//STEP1   EXEC COPYFILE
```

**C'est tout ! 🎉**

Le système va "insérer" le contenu de la PROC dans ton job.

### Où Stocker les PROCS ?

**Les PROCS sont stockées dans des bibliothèques systèmes :**

**1. PROCLIB système (fournie par IBM) :**
```
SYS1.PROCLIB
```

**2. PROCLIB personnalisée (ta compagnie) :**
```
PROD.PROCLIB
USER.PROCLIB
```

**Pour que JES trouve ta PROC, elle doit être dans une PROCLIB référencée.**

**Vérifier les PROCLIB actives :**

```jcl
//SHOWPROC JOB ...
//STEP1 EXEC PGM=IEFBR14
/*JOBPARM PROCLIB=PROD.PROCLIB
```

### PROC Inline vs Cataloged

**PROC Cataloged (normale) :**
- Stockée dans une PROCLIB
- Utilisée par plusieurs jobs
- Standard en production

```jcl
//STEP1 EXEC COPYFILE  ← PROC cataloguée
```

---

**PROC Inline (dans le JOB) :**
- Définie directement dans le job
- Utilisée seulement dans ce job
- Pratique pour tester

```jcl
//TESTJOB JOB (ACCT),'TEST PROC',CLASS=A,MSGCLASS=X
//MYPROC  PROC
//STEP1   EXEC PGM=IEFBR14
//DD1     DD DSN=TEST.FILE,DISP=SHR
//        PEND
//RUN     EXEC MYPROC
```

### PROC avec Plusieurs Steps

**Les PROCS peuvent contenir plusieurs steps.**

**Exemple : Compile + Link COBOL**

```jcl
//COBLNK  PROC
//*
//* Step 1: Compile COBOL
//*
//COMPILE EXEC PGM=IGYCRCTL
//STEPLIB  DD DSN=IGY.SIGYCOMP,DISP=SHR
//SYSPRINT DD SYSOUT=*
//SYSLIN   DD DSN=&&LOADSET,DISP=(MOD,PASS),
//            UNIT=SYSDA,SPACE=(CYL,(1,1))
//SYSUT1   DD UNIT=SYSDA,SPACE=(CYL,(1,1))
//SYSUT2   DD UNIT=SYSDA,SPACE=(CYL,(1,1))
//SYSUT3   DD UNIT=SYSDA,SPACE=(CYL,(1,1))
//SYSUT4   DD UNIT=SYSDA,SPACE=(CYL,(1,1))
//SYSUT5   DD UNIT=SYSDA,SPACE=(CYL,(1,1))
//SYSUT6   DD UNIT=SYSDA,SPACE=(CYL,(1,1))
//SYSUT7   DD UNIT=SYSDA,SPACE=(CYL,(1,1))
//SYSIN    DD DSN=SOURCE.COBOL(PROG1),DISP=SHR
//*
//* Step 2: Link-edit
//*
//LKED    EXEC PGM=IEWL
//SYSLIB   DD DSN=CEE.SCEELKED,DISP=SHR
//SYSPRINT DD SYSOUT=*
//SYSLMOD  DD DSN=LOADLIB.PROD(PROG1),DISP=SHR
//SYSUT1   DD UNIT=SYSDA,SPACE=(CYL,(1,1))
//SYSLIN   DD DSN=&&LOADSET,DISP=(OLD,DELETE)
//         PEND
```

**Utilisation :**

```jcl
//COMPLINK JOB (ACCT),'COMPILE AND LINK',CLASS=A,MSGCLASS=X
//BUILD    EXEC COBLNK
```

**Le job exécute automatiquement compile + link en séquence.**

### Override DD Statements

**Tu peux "override" (remplacer) des DD dans une PROC.**

**PROC originale :**

```jcl
//COPYFILE PROC
//COPY     EXEC PGM=IEBGENER
//SYSUT1   DD DSN=INPUT.FILE,DISP=SHR
//SYSUT2   DD DSN=OUTPUT.FILE,DISP=(NEW,CATLG)
//         PEND
```

**Dans ton JOB, tu override les DD :**

```jcl
//COPYJOB JOB (ACCT),'COPY WITH OVERRIDE',CLASS=A,MSGCLASS=X
//STEP1   EXEC COPYFILE
//COPY.SYSUT1 DD DSN=MY.INPUT.FILE,DISP=SHR  ← Override input
//COPY.SYSUT2 DD DSN=MY.OUTPUT.FILE,DISP=(NEW,CATLG)  ← Override output
```

**Syntaxe :** `//stepname.ddname`

**Résultat :** La PROC utilise TES fichiers au lieu de ceux par défaut.

### Ajouter des DD à une PROC

**Tu peux aussi AJOUTER des DD qui ne sont pas dans la PROC.**

```jcl
//STEP1   EXEC COPYFILE
//COPY.SYSPRINT DD SYSOUT=*  ← Ajoute un DD
```

### Cas Réel : PROC de Backup

**PROC standard pour backup quotidien :**

```jcl
//BACKUP  PROC
//COPY    EXEC PGM=IDCAMS
//SYSPRINT DD SYSOUT=*
//SYSIN    DD *
  REPRO INFILE(INDD) OUTFILE(OUTDD)
/*
//         PEND
```

**Utilisation pour différents fichiers :**

```jcl
//BACKUP1 JOB (ACCT),'BACKUP CUSTOMERS',CLASS=A,MSGCLASS=X
//STEP1   EXEC BACKUP
//INDD    DD DSN=PROD.CUSTOMERS,DISP=SHR
//OUTDD   DD DSN=BACKUP.CUSTOMERS(+1),
//           DISP=(NEW,CATLG),
//           UNIT=TAPE
```

```jcl
//BACKUP2 JOB (ACCT),'BACKUP ORDERS',CLASS=A,MSGCLASS=X
//STEP1   EXEC BACKUP
//INDD    DD DSN=PROD.ORDERS,DISP=SHR
//OUTDD   DD DSN=BACKUP.ORDERS(+1),
//           DISP=(NEW,CATLG),
//           UNIT=TAPE
```

**Même PROC, fichiers différents. Simple. Puissant.**

### PROC avec EXEC Statement Override

**Tu peux override des paramètres EXEC.**

**PROC :**

```jcl
//RUNPROG PROC
//STEP1   EXEC PGM=CUSTPROG,
//             PARM='MODE=PROD'
//         PEND
```

**Override du PARM :**

```jcl
//TESTJOB JOB ...
//RUN     EXEC RUNPROG
//STEP1   EXEC PARM='MODE=TEST'  ← Override PARM
```

### Règles Importantes des PROCS

**1. Les DD overrides doivent être APRÈS l'EXEC de la PROC**

❌ **FAUX :**
```jcl
//STEP1.DD1 DD ...  ← Override AVANT l'EXEC
//STEP1 EXEC MYPROC
```

✅ **CORRECT :**
```jcl
//STEP1 EXEC MYPROC
//STEP1.DD1 DD ...  ← Override APRÈS l'EXEC
```

---

**2. Tu ne peux pas supprimer un DD d'une PROC**

Tu peux seulement override ou ajouter, pas supprimer.

---

**3. Tu dois utiliser `stepname.ddname` pour override**

```jcl
//STEP1 EXEC MYPROC
//STEP1.DD1 DD ...  ← stepname.ddname
```

### Nested PROCS (PROCS imbriquées)

**Une PROC peut appeler une autre PROC !**

**PROC 1 : Compile COBOL**

```jcl
//COBCOMP PROC
//COMP    EXEC PGM=IGYCRCTL
//SYSIN   DD DSN=SOURCE.COBOL(PROG),DISP=SHR
//        PEND
```

**PROC 2 : Compile + Link**

```jcl
//COBLNK  PROC
//COMPILE EXEC COBCOMP  ← Appelle PROC 1
//LINK    EXEC PGM=IEWL
//SYSLIN  DD DSN=&&LOADSET,DISP=(OLD,DELETE)
//        PEND
```

**JOB :**

```jcl
//BUILD JOB ...
//STEP1 EXEC COBLNK  ← Appelle PROC 2 qui appelle PROC 1
```

**Limite :** Maximum 15 niveaux d'imbrication (largement suffisant).

### PROC pour Environnements Multiples

**Même code, plusieurs environnements : DEV, TEST, PROD**

**PROC générique :**

```jcl
//DEPLOY  PROC ENV=PROD
//COPY    EXEC PGM=IEBGENER
//SYSUT1  DD DSN=SOURCE.PROGRAM,DISP=SHR
//SYSUT2  DD DSN=&ENV..LOADLIB(PROG),DISP=SHR
//        PEND
```

**Utilisation :**

```jcl
// DEV
//STEP1 EXEC DEPLOY,ENV=DEV

// TEST
//STEP1 EXEC DEPLOY,ENV=TEST

// PROD
//STEP1 EXEC DEPLOY,ENV=PROD
```

**On verra ça en détail au Chapitre 16 (Paramètres Symboliques).**

### Erreurs Courantes avec PROCS

**ERREUR #1 : PROC NOT FOUND**

```
IEF653I SUBSTITUTION JCL - PROCNAME NOT FOUND
```

**Cause :** La PROC n'existe pas dans les PROCLIB  
**Solution :** Vérifie le nom ou ajoute la PROCLIB au JCL

---

**ERREUR #2 : Override Syntax Error**

```jcl
//STEP1.DD1 DD ...  ← Override AVANT l'EXEC = ERREUR
//STEP1 EXEC MYPROC
```

**Solution :** Mets l'override APRÈS l'EXEC

---

**ERREUR #3 : Missing Stepname**

```jcl
//STEP1 EXEC MYPROC
//DD1 DD ...  ← Manque le stepname
```

**Solution :**
```jcl
//STEP1.DD1 DD ...  ← Ajoute le stepname
```

### Exercice Pratique : PROCS

**Crée une PROC qui :**
1. Trie un fichier (PGM=SORT)
2. Copie le résultat vers un dataset de sortie
3. Utilise-la pour trier 2 fichiers différents

**Solution :**

**PROC :**

```jcl
//SORTCOPY PROC
//SORT    EXEC PGM=SORT
//SYSOUT   DD SYSOUT=*
//SORTIN   DD DSN=INPUT.FILE,DISP=SHR
//SORTOUT  DD DSN=OUTPUT.FILE,DISP=(NEW,CATLG),
//            UNIT=SYSDA,SPACE=(CYL,(10,2))
//SYSIN    DD *
  SORT FIELDS=(1,10,CH,A)
/*
//         PEND
```

**Utilisation pour fichier 1 :**

```jcl
//SORTJOB1 JOB (ACCT),'SORT FILE 1',CLASS=A,MSGCLASS=X
//STEP1    EXEC SORTCOPY
//SORT.SORTIN  DD DSN=TRANSACTIONS.JAN,DISP=SHR
//SORT.SORTOUT DD DSN=SORTED.TRANSACTIONS.JAN,DISP=(NEW,CATLG)
```

**Utilisation pour fichier 2 :**

```jcl
//SORTJOB2 JOB (ACCT),'SORT FILE 2',CLASS=A,MSGCLASS=X
//STEP1    EXEC SORTCOPY
//SORT.SORTIN  DD DSN=TRANSACTIONS.FEB,DISP=SHR
//SORT.SORTOUT DD DSN=SORTED.TRANSACTIONS.FEB,DISP=(NEW,CATLG)
```

---

## 16. Paramètres Symboliques

### Qu'est-ce qu'un Paramètre Symbolique ?

**Paramètre Symbolique = Variable dans une PROC.**

**Le problème sans paramètres symboliques :**

Tu as une PROC pour backup, mais tu veux :
- Changer le nom du fichier d'entrée
- Changer le nom du fichier de sortie
- Changer l'unité (TAPE vs DASD)

Sans paramètres symboliques, tu dois créer 3 PROCS différentes ou override tous les DD.

**Avec paramètres symboliques :**

Tu crées UNE PROC avec des variables, tu passes les valeurs quand tu l'appelles.

```jcl
//BACKUP PROC INFILE=,OUTFILE=,UNIT=TAPE
//COPY   EXEC PGM=IEBGENER
//SYSUT1 DD DSN=&INFILE,DISP=SHR
//SYSUT2 DD DSN=&OUTFILE,DISP=(NEW,CATLG),UNIT=&UNIT
//       PEND
```

**Appel :**

```jcl
//STEP1 EXEC BACKUP,
//      INFILE=PROD.CUSTOMERS,
//      OUTFILE=BACKUP.CUSTOMERS,
//      UNIT=TAPE
```

**Les valeurs sont substituées automatiquement !**

### Syntaxe des Paramètres Symboliques

**Déclaration dans la PROC :**

```jcl
//PROCNAME PROC param1=default1,param2=default2
```

**Utilisation dans la PROC :**

```jcl
&param1
&param2
```

**Le `&` indique que c'est une variable.**

### Exemple Simple

**PROC avec un paramètre :**

```jcl
//COPYFILE PROC DSN=DEFAULT.FILE
//COPY     EXEC PGM=IEBGENER
//SYSUT1   DD DSN=&DSN,DISP=SHR
//SYSUT2   DD DSN=OUTPUT.FILE,DISP=(NEW,CATLG)
//         PEND
```

**Appel avec valeur :**

```jcl
//STEP1 EXEC COPYFILE,DSN=MY.INPUT.FILE
```

**Le système remplace `&DSN` par `MY.INPUT.FILE`.**

### Valeurs par Défaut

**Tu peux définir des valeurs par défaut.**

```jcl
//BACKUP PROC ENV=PROD,UNIT=TAPE
//COPY   EXEC PGM=IEBGENER
//SYSUT2 DD DSN=&ENV..BACKUP.FILE,UNIT=&UNIT
//       PEND
```

**Si tu n'override pas, les valeurs par défaut sont utilisées :**

```jcl
//STEP1 EXEC BACKUP  ← ENV=PROD, UNIT=TAPE (défaut)
```

**Si tu override :**

```jcl
//STEP1 EXEC BACKUP,ENV=TEST,UNIT=DASD
```

### Paramètres Multiples

**Tu peux avoir plusieurs paramètres.**

```jcl
//DEPLOY PROC ENV=,PGM=,LIB=
//COPY   EXEC PGM=IEBGENER
//SYSUT1 DD DSN=SOURCE.&PGM,DISP=SHR
//SYSUT2 DD DSN=&ENV..&LIB(&PGM),DISP=SHR
//       PEND
```

**Appel :**

```jcl
//STEP1 EXEC DEPLOY,
//      ENV=PROD,
//      PGM=PAYROLL,
//      LIB=LOADLIB
```

**Résultat :**
```jcl
//SYSUT1 DD DSN=SOURCE.PAYROLL,DISP=SHR
//SYSUT2 DD DSN=PROD.LOADLIB(PAYROLL),DISP=SHR
```

### Substitution dans les Noms de Datasets

**Utilise `&` pour construire des noms dynamiquement.**

**Exemple : Datasets par environnement**

```jcl
//RUNBATCH PROC ENV=PROD
//STEP1    EXEC PGM=BATCHPGM
//INPUT    DD DSN=&ENV..CUSTOMER.MASTER,DISP=SHR
//OUTPUT   DD DSN=&ENV..REPORT.DAILY,DISP=(NEW,CATLG)
//         PEND
```

**DEV :**
```jcl
//STEP1 EXEC RUNBATCH,ENV=DEV
// → DSN=DEV.CUSTOMER.MASTER
// → DSN=DEV.REPORT.DAILY
```

**PROD :**
```jcl
//STEP1 EXEC RUNBATCH,ENV=PROD
// → DSN=PROD.CUSTOMER.MASTER
// → DSN=PROD.REPORT.DAILY
```

### Concatenation avec `.`

**Pour éviter les ambiguïtés, utilise `..` pour séparer.**

**Exemple :**

```jcl
&ENV..CUSTOMER.MASTER
↑   ↑
│   └─ Le point fait partie du DSN
└───── Double point pour séparer la variable
```

**Sans le double point :**
```jcl
&ENV.CUSTOMER.MASTER  ← Erreur ! Cherche une variable ENV.CUSTOMER
```

### Paramètres dans PARM

**Tu peux passer des paramètres au programme.**

```jcl
//RUNPROG PROC MODE=PROD
//STEP1   EXEC PGM=CUSTPROG,PARM='&MODE'
//        PEND
```

**Appel :**

```jcl
//STEP1 EXEC RUNPROG,MODE=TEST
// → PARM='TEST'
```

### Paramètres dans EXEC

**Tu peux utiliser des paramètres pour le nom du programme.**

```jcl
//RUNANY PROC PGM=IEFBR14
//STEP1  EXEC PGM=&PGM
//       PEND
```

**Appel :**

```jcl
//STEP1 EXEC RUNANY,PGM=SORT
// → EXEC PGM=SORT
```

### SET Statement (JCL moderne)

**Depuis JCL moderne, tu peux définir des symboles avec SET.**

```jcl
//MYJOB JOB ...
//  SET ENV=PROD
//  SET HLQ=FINANCE
//STEP1 EXEC PGM=BATCHPGM
//INPUT DD DSN=&ENV..&HLQ..CUSTOMER.MASTER,DISP=SHR
```

**SET permet de définir des variables au niveau du JOB, pas juste dans les PROCS.**

### Cas Réel : PROC Multi-Environnement

**PROC pour compiler + déployer selon l'environnement :**

```jcl
//COMPILE PROC ENV=DEV,PGM=,SRC=SOURCE.COBOL,LIB=LOADLIB
//*
//* Step 1: Compile
//*
//COMP    EXEC PGM=IGYCRCTL
//STEPLIB  DD DSN=IGY.SIGYCOMP,DISP=SHR
//SYSPRINT DD SYSOUT=*
//SYSLIN   DD DSN=&&LOADSET,DISP=(MOD,PASS)
//SYSIN    DD DSN=&SRC(&PGM),DISP=SHR
//*
//* Step 2: Link
//*
//LKED    EXEC PGM=IEWL
//SYSLIB   DD DSN=CEE.SCEELKED,DISP=SHR
//SYSPRINT DD SYSOUT=*
//SYSLMOD  DD DSN=&ENV..&LIB(&PGM),DISP=SHR
//SYSLIN   DD DSN=&&LOADSET,DISP=(OLD,DELETE)
//         PEND
```

**Utilisation DEV :**

```jcl
//DEVBUILD JOB ...
//BUILD EXEC COMPILE,ENV=DEV,PGM=PAYROLL
// → Compile vers DEV.LOADLIB(PAYROLL)
```

**Utilisation PROD :**

```jcl
//PRODBUILD JOB ...
//BUILD EXEC COMPILE,ENV=PROD,PGM=PAYROLL
// → Compile vers PROD.LOADLIB(PAYROLL)
```

**Même PROC, plusieurs environnements. Zero duplication.**

### Règles des Paramètres Symboliques

**1. Les noms doivent commencer par une lettre**

✅ **CORRECT :**
```jcl
//PROC1 PROC ENV=,FILE1=
```

❌ **FAUX :**
```jcl
//PROC1 PROC 1ENV=,2FILE=  ← Commence par un chiffre
```

---

**2. Maximum 8 caractères**

✅ **CORRECT :**
```jcl
//PROC1 PROC FILENAME=
```

❌ **FAUX :**
```jcl
//PROC1 PROC VERYLONGNAME=  ← Plus de 8 caractères
```

---

**3. Toujours utiliser `&` pour référencer**

✅ **CORRECT :**
```jcl
DD DSN=&FILENAME
```

❌ **FAUX :**
```jcl
DD DSN=FILENAME  ← Sans &, c'est une chaîne littérale
```

### Paramètres NULL

**Si un paramètre n'a pas de valeur, il disparaît.**

```jcl
//MYPROC PROC EXTRA=
//DD1 DD DSN=FILE.DATA&EXTRA,DISP=SHR
//     PEND
```

**Si EXTRA est vide :**
```jcl
//STEP1 EXEC MYPROC
// → DD DSN=FILE.DATA
```

**Si EXTRA a une valeur :**
```jcl
//STEP1 EXEC MYPROC,EXTRA=.BACKUP
// → DD DSN=FILE.DATA.BACKUP
```

### Override vs Paramètres Symboliques

**Quelle différence ?**

**Override DD :**
- Remplace TOUT le DD statement
- Plus flexible mais plus verbeux

```jcl
//STEP1 EXEC MYPROC
//STEP1.DD1 DD DSN=NEW.FILE,DISP=SHR,UNIT=SYSDA,SPACE=(CYL,(10))
```

**Paramètre Symbolique :**
- Change seulement certaines valeurs
- Plus simple et plus lisible

```jcl
//STEP1 EXEC MYPROC,DSN=NEW.FILE
```

**Utilise des paramètres symboliques quand tu changes souvent les mêmes valeurs.**

### Erreurs Courantes

**ERREUR #1 : Variable non définie**

```
IEF653I SUBSTITUTION JCL - &VARNAME NOT DEFINED
```

**Cause :** Tu utilises `&VARNAME` mais elle n'existe pas dans la PROC  
**Solution :** Définis-la dans le PROC statement

---

**ERREUR #2 : Mauvaise concatenation**

```jcl
DD DSN=&ENV.CUSTOMER.MASTER  ← Cherche &ENV.CUSTOMER
```

**Solution :**
```jcl
DD DSN=&ENV..CUSTOMER.MASTER  ← Double point
```

---

**ERREUR #3 : Quotes manquantes**

```jcl
PARM=&MODE  ← Si MODE contient des espaces, ça plante
```

**Solution :**
```jcl
PARM='&MODE'  ← Toujours mettre des quotes
```

### Exercice Pratique : Paramètres Symboliques

**Crée une PROC qui :**
1. Accepte un paramètre pour le nom du programme COBOL
2. Accepte un paramètre pour l'environnement (DEV/TEST/PROD)
3. Compile le programme vers le loadlib correct

**Solution :**

```jcl
//COBCOMP PROC PGM=,ENV=DEV
//*
//* Compile COBOL
//*
//COMP    EXEC PGM=IGYCRCTL
//STEPLIB  DD DSN=IGY.SIGYCOMP,DISP=SHR
//SYSPRINT DD SYSOUT=*
//SYSLIN   DD DSN=&&LOADSET,DISP=(MOD,PASS)
//SYSIN    DD DSN=SOURCE.COBOL(&PGM),DISP=SHR
//*
//* Link-edit
//*
//LKED    EXEC PGM=IEWL
//SYSLIB   DD DSN=CEE.SCEELKED,DISP=SHR
//SYSPRINT DD SYSOUT=*
//SYSLMOD  DD DSN=&ENV..LOADLIB(&PGM),DISP=SHR
//SYSLIN   DD DSN=&&LOADSET,DISP=(OLD,DELETE)
//         PEND
```

**Utilisation :**

```jcl
//COMPDEV JOB (ACCT),'COMPILE DEV',CLASS=A,MSGCLASS=X
//BUILD   EXEC COBCOMP,PGM=PAYROLL,ENV=DEV

//COMPPROD JOB (ACCT),'COMPILE PROD',CLASS=A,MSGCLASS=X
//BUILD   EXEC COBCOMP,PGM=PAYROLL,ENV=PROD
```

---

## 17. Conditional Processing

### Qu'est-ce que le Conditional Processing ?

**Conditional Processing = Contrôler l'exécution des steps selon des conditions.**

**Le problème sans conditional processing :**

Tu as un job avec 5 steps :
1. Backup
2. Process
3. Generate report
4. Send email success
5. Send email failure

**Sans conditions :**
- Si le backup échoue, les 4 autres steps s'exécutent quand même 😱
- Tu envoies un email "success" même si tout a planté 🤦

**Avec conditional processing :**
- Si le backup échoue → arrête tout et envoie email d'erreur
- Si le processing réussit → envoie email success
- **Contrôle intelligent de l'exécution**

### Return Codes (Rappel)

**Chaque step retourne un Return Code (RC) :**

| RC | Signification |
|----|---------------|
| 0 | Succès parfait |
| 4 | Warning (généralement OK) |
| 8 | Erreur |
| 12 | Erreur grave |
| 16+ | Erreur fatale |

**Le conditional processing se base sur ces RC.**

### COND Parameter - La Méthode Classique

**COND = CONDition**

**Syntaxe :**
```jcl
//STEP EXEC PGM=...,COND=(value,operator,stepname)
```

**Logique (ATTENTION, c'est contre-intuitif) :**

**Si la condition est VRAIE → SKIP le step**
**Si la condition est FAUSSE → EXECUTE le step**

**Oui, c'est inversé ! 🤯**

### Opérateurs COND

| Opérateur | Signification |
|-----------|---------------|
| **GT** | Greater Than (>) |
| **GE** | Greater or Equal (>=) |
| **EQ** | Equal (=) |
| **NE** | Not Equal (≠) |
| **LT** | Less Than (<) |
| **LE** | Less or Equal (<=) |

### Exemple Simple : Skip si Erreur

**"Exécute STEP2 seulement si STEP1 a réussi (RC=0)"**

```jcl
//STEP1 EXEC PGM=BACKUP
//STEP2 EXEC PGM=PROCESS,COND=(0,NE,STEP1)
```

**Traduction :**
```
Si (RC de STEP1 ≠ 0) alors SKIP STEP2
Sinon EXECUTE STEP2
```

**En d'autres mots :** STEP2 tourne seulement si STEP1 a retourné 0.

### Exemple : Skip si Succès

**"Exécute STEP3 (cleanup) seulement si STEP2 a échoué"**

```jcl
//STEP2 EXEC PGM=MAINPROCESS
//STEP3 EXEC PGM=CLEANUP,COND=(0,EQ,STEP2)
```

**Traduction :**
```
Si (RC de STEP2 = 0) alors SKIP STEP3
Sinon EXECUTE STEP3
```

**Cleanup tourne seulement en cas d'erreur.**

### Exemple : Exécuter si RC < 8

**"Exécute STEP4 si STEP3 a retourné moins de 8 (0-7)"**

```jcl
//STEP3 EXEC PGM=DATAPROCESS
//STEP4 EXEC PGM=REPORT,COND=(8,LE,STEP3)
```

**Traduction :**
```
Si (RC de STEP3 >= 8) alors SKIP STEP4
Sinon EXECUTE STEP4
```

**STEP4 tourne seulement si RC de STEP3 est 0-7 (succès ou warning).**

### COND au Niveau JOB

**Tu peux mettre COND sur la JOB card pour tout le job.**

```jcl
//MYJOB JOB (ACCT),'CONDITIONAL JOB',
//      CLASS=A,MSGCLASS=X,
//      COND=(8,LT)
```

**Traduction :**
```
Si un step retourne RC < 8, SKIP tous les steps suivants
```

**Ça arrête le job dès qu'un RC est inférieur à 8.**

### COND=EVEN et COND=ONLY

**Deux valeurs spéciales :**

**COND=EVEN**
- Exécute le step MÊME SI un step précédent a ABEND
- Utilisé pour cleanup ou notification

```jcl
//STEP1 EXEC PGM=MAINPROG
//STEP2 EXEC PGM=CLEANUP,COND=EVEN
```

**Si STEP1 plante (ABEND), STEP2 tourne quand même.**

---

**COND=ONLY**
- Exécute le step SEULEMENT SI un step précédent a ABEND
- Utilisé pour error handling

```jcl
//STEP1 EXEC PGM=MAINPROG
//STEP2 EXEC PGM=ERRORHANDLER,COND=ONLY
```

**STEP2 tourne seulement si STEP1 a planté.**

### IF/THEN/ELSE/ENDIF - La Méthode Moderne

**Depuis z/OS 1.13, il y a une syntaxe plus claire : IF/THEN/ELSE**

**C'est BEAUCOUP plus lisible que COND !**

### Syntaxe IF/THEN/ELSE

```jcl
//IF1 IF (condition) THEN
//stepname EXEC PGM=...
//ENDIF
```

**Ou avec ELSE :**

```jcl
//IF1 IF (condition) THEN
//stepname1 EXEC PGM=...
//ELSE
//stepname2 EXEC PGM=...
//ENDIF
```

### Conditions IF

**Syntaxe des conditions :**

```jcl
stepname.RC operator value
```

**Opérateurs :**
- `=` Equal
- `¬=` ou `^=` Not Equal
- `>` Greater Than
- `>=` Greater or Equal
- `<` Less Than
- `<=` Less or Equal

### Exemple : IF/THEN Simple

**"Si STEP1 réussit, exécute STEP2"**

```jcl
//STEP1 EXEC PGM=BACKUP
//IF1 IF (STEP1.RC = 0) THEN
//STEP2 EXEC PGM=PROCESS
//ENDIF
```

**BEAUCOUP plus clair que COND !**

### Exemple : IF/THEN/ELSE

**"Si STEP1 réussit, envoie email success, sinon envoie email error"**

```jcl
//STEP1 EXEC PGM=MAINPROCESS
//IF1 IF (STEP1.RC = 0) THEN
//SUCCESS EXEC PGM=SENDMAIL,PARM='STATUS=OK'
//ELSE
//FAILURE EXEC PGM=SENDMAIL,PARM='STATUS=ERROR'
//ENDIF
```

### Conditions Multiples (AND/OR)

**AND (ET) :**

```jcl
//IF1 IF (STEP1.RC = 0 AND STEP2.RC = 0) THEN
//SUCCESS EXEC PGM=REPORT
//ENDIF
```

**OR (OU) :**

```jcl
//IF1 IF (STEP1.RC > 4 OR STEP2.RC > 4) THEN
//ERROR EXEC PGM=CLEANUP
//ENDIF
```

### NOT (Négation)

```jcl
//IF1 IF (NOT STEP1.RC = 0) THEN
//ERROR EXEC PGM=ERRORHANDLER
//ENDIF
```

**Équivalent à :**
```jcl
//IF1 IF (STEP1.RC ^= 0) THEN
```

### Parenthèses pour Grouper

```jcl
//IF1 IF ((STEP1.RC = 0 AND STEP2.RC = 0) OR STEP3.RC = 0) THEN
//CONTINUE EXEC PGM=NEXTPROG
//ENDIF
```

### ABENDED Condition

**Tester si un step a ABEND :**

```jcl
//STEP1 EXEC PGM=MAINPROG
//IF1 IF (STEP1.ABENDED) THEN
//CLEANUP EXEC PGM=ERRORCLEANUP
//ENDIF
```

### RUN Condition

**Tester si un step a tourné :**

```jcl
//IF1 IF (STEP1.RUN) THEN
//STEP2 EXEC PGM=FOLLOWUP
//ENDIF
```

### Cas Réel : Backup avec Notification

**Job complet avec gestion d'erreur :**

```jcl
//BACKUP JOB (ACCT),'DAILY BACKUP WITH NOTIFICATION',
//       CLASS=A,MSGCLASS=X
//*
//* Step 1: Backup VSAM
//*
//BACKUP  EXEC PGM=IDCAMS
//SYSPRINT DD SYSOUT=*
//SYSIN    DD *
  REPRO INFILE(PROD) OUTFILE(BACKUP)
/*
//PROD   DD DSN=PROD.CUSTOMER.MASTER,DISP=SHR
//BACKUP DD DSN=BACKUP.CUSTOMER.DAILY(+1),
//          DISP=(NEW,CATLG),
//          UNIT=TAPE
//*
//* Step 2: If backup succeeded, send success email
//*
//IF1 IF (BACKUP.RC = 0) THEN
//NOTIFY1 EXEC PGM=SENDMAIL,PARM='BACKUP SUCCESS'
//ENDIF
//*
//* Step 3: If backup failed, send error email
//*
//IF2 IF (BACKUP.RC > 0) THEN
//NOTIFY2 EXEC PGM=SENDMAIL,PARM='BACKUP FAILED'
//CLEANUP EXEC PGM=ERRORLOG,PARM='BACKUP'
//ENDIF
```

### Cas Réel : Processing Pipeline

**Pipeline avec plusieurs étapes dépendantes :**

```jcl
//PIPELINE JOB (ACCT),'DATA PIPELINE',CLASS=A,MSGCLASS=X
//*
//* Step 1: Extract data
//*
//EXTRACT EXEC PGM=EXTRACTPGM
//*
//* Step 2: Only if extract succeeded, transform data
//*
//IF1 IF (EXTRACT.RC = 0) THEN
//TRANSFORM EXEC PGM=TRANSFORMPGM
//ENDIF
//*
//* Step 3: Only if transform succeeded, load data
//*
//IF2 IF (EXTRACT.RC = 0 AND TRANSFORM.RC = 0) THEN
//LOAD EXEC PGM=LOADPGM
//ENDIF
//*
//* Step 4: Generate report if all succeeded
//*
//IF3 IF (EXTRACT.RC = 0 AND 
//        TRANSFORM.RC = 0 AND 
//        LOAD.RC = 0) THEN
//REPORT EXEC PGM=REPORTPGM
//ENDIF
//*
//* Step 5: Error handling if anything failed
//*
//IF4 IF (EXTRACT.RC > 0 OR 
//        TRANSFORM.RC > 0 OR 
//        LOAD.RC > 0) THEN
//ROLLBACK EXEC PGM=ROLLBACKPGM
//ALERT EXEC PGM=SENDMAIL,PARM='PIPELINE FAILED'
//ENDIF
```

### Nested IF (IF imbriqués)

**Tu peux imbriquer des IF/THEN/ELSE :**

```jcl
//IF1 IF (STEP1.RC = 0) THEN
//  IF2 IF (STEP2.RC = 0) THEN
//    SUCCESS EXEC PGM=REPORT
//  ELSE
//    PARTIAL EXEC PGM=PARTIALREPORT
//  ENDIF
//ELSE
//  ERROR EXEC PGM=ERRORHANDLER
//ENDIF
```

### COND vs IF/THEN : Lequel Utiliser ?

**COND (méthode classique) :**
- ✅ Compatible avec tous les systèmes (même anciens)
- ❌ Syntaxe contre-intuitive
- ❌ Difficile à lire

**IF/THEN/ELSE (méthode moderne) :**
- ✅ Syntaxe claire et intuitive
- ✅ Facile à lire et maintenir
- ❌ Requiert z/OS 1.13+ (2011)

**Recommandation : Utilise IF/THEN/ELSE si possible.**

### Erreurs Courantes

**ERREUR #1 : COND inversé**

```jcl
//STEP2 EXEC PGM=...,COND=(0,EQ,STEP1)  ← Skip si RC=0 !
```

**Beaucoup de gens se trompent parce que la logique COND est inversée.**

**Solution :** Utilise IF/THEN qui est plus clair.

---

**ERREUR #2 : IF sans ENDIF**

```jcl
//IF1 IF (STEP1.RC = 0) THEN
//STEP2 EXEC PGM=...
// ← Manque ENDIF !
```

**Solution :** Toujours fermer avec ENDIF.

---

**ERREUR #3 : Référencer un step qui n'existe pas**

```jcl
//IF1 IF (STEP99.RC = 0) THEN  ← STEP99 n'existe pas
```

**Solution :** Vérifie les noms des steps.

### Exercice Pratique : Conditional Processing

**Crée un job qui :**
1. Trie un fichier (STEP1)
2. Si le tri réussit, copie le résultat (STEP2)
3. Si le tri réussit, génère un rapport (STEP3)
4. Si le tri échoue, envoie une alerte (STEP4)

**Solution :**

```jcl
//SORTJOB JOB (ACCT),'CONDITIONAL SORT',CLASS=A,MSGCLASS=X
//*
//* Step 1: Sort data
//*
//SORT    EXEC PGM=SORT
//SYSOUT   DD SYSOUT=*
//SORTIN   DD DSN=INPUT.TRANSACTIONS,DISP=SHR
//SORTOUT  DD DSN=SORTED.TRANSACTIONS,
//            DISP=(NEW,CATLG),
//            SPACE=(CYL,(10,2))
//SYSIN    DD *
  SORT FIELDS=(1,10,CH,A)
/*
//*
//* Step 2: Copy sorted file (if sort succeeded)
//*
//IF1 IF (SORT.RC = 0) THEN
//COPY    EXEC PGM=IEBGENER
//SYSPRINT DD SYSOUT=*
//SYSUT1   DD DSN=SORTED.TRANSACTIONS,DISP=SHR
//SYSUT2   DD DSN=ARCHIVE.TRANSACTIONS,DISP=(NEW,CATLG)
//SYSIN    DD DUMMY
//ENDIF
//*
//* Step 3: Generate report (if sort succeeded)
//*
//IF2 IF (SORT.RC = 0) THEN
//REPORT  EXEC PGM=REPORTPGM
//INPUT    DD DSN=SORTED.TRANSACTIONS,DISP=SHR
//OUTPUT   DD DSN=REPORT.DAILY,DISP=(NEW,CATLG)
//ENDIF
//*
//* Step 4: Send alert (if sort failed)
//*
//IF3 IF (SORT.RC > 0) THEN
//ALERT   EXEC PGM=SENDMAIL,PARM='SORT FAILED'
//ENDIF
```

---

## 18. JES2/JES3 - Job Entry Subsystem

### Qu'est-ce que JES ?

**JES = Job Entry Subsystem**

**JES est le "chef d'orchestre" du mainframe.**

**Sans JES, ton JCL est juste du texte. JES le transforme en exécution.**

**Analogie :**
- Ton JCL = Partition musicale
- JES = Chef d'orchestre qui dirige l'exécution
- z/OS = Orchestre qui joue

### Que Fait JES ?

**JES gère TOUT le cycle de vie des jobs :**

1. **INPUT** - Reçoit le JCL (depuis terminal, fichier, réseau)
2. **QUEUE** - Met le job en attente
3. **SCHEDULE** - Décide quand le job tourne
4. **EXECUTE** - Lance le job
5. **OUTPUT** - Gère les résultats (SYSOUT)
6. **PURGE** - Nettoie après exécution

**JES est TOUJOURS actif. 24/7. 365 jours/an.**

### JES2 vs JES3

**Il y a 2 versions de JES :**

**JES2 (le plus courant)**
- ✅ Simple et rapide
- ✅ Utilisé par 80%+ des installations
- ✅ Bon pour single-system ou petit sysplex
- ❌ Moins de features avancées

**JES3 (plus rare)**
- ✅ Features avancées
- ✅ Meilleur pour sysplex complexes
- ✅ Job scheduling sophistiqué
- ❌ Plus complexe à gérer

**La plupart des banques utilisent JES2.**

**On va se concentrer sur JES2 dans ce guide.**

### Le Cycle de Vie d'un Job

**Étape par étape :**

1. **Tu soumets le JCL**
```bash
SUBMIT 'MY.JCL(MYJOB)'
```

2. **JES2 reçoit le JCL**
- Analyse la syntaxe
- Assigne un Job ID (ex: JOB12345)
- Met le job en INPUT queue

3. **JES2 met en queue**
- Regarde la CLASS du job
- Détermine la priorité
- Place dans la bonne queue

4. **JES2 schedule l'exécution**
- Attend les ressources (DASD, tape, etc.)
- Vérifie les dépendances
- Lance quand tout est prêt

5. **z/OS exécute le job**
- Alloue les datasets
- Lance les programmes
- Capture les outputs

6. **JES2 gère la sortie**
- Collecte SYSOUT
- Stocke les résultats
- Rend accessible via SDSF

7. **JES2 purge le job**
- Après X temps (configurable)
- Libère les ressources

### Job States (États d'un Job)

**Un job passe par plusieurs états :**

| État | Code | Description |
|------|------|-------------|
| **INPUT** | - | Job reçu, pas encore schedulé |
| **HOLD** | H | Job en attente (volontaire) |
| **ACTIVE** | A | Job en cours d'exécution |
| **OUTPUT** | O | Job terminé, output disponible |
| **COMPLETED** | C | Job fini et purgé |

### Job ID

**Chaque job reçoit un Job ID unique.**

**Format :** `JOBxxxxx` ou `STC xxxxx` ou `TSUxxxxx`

**Exemples :**
- `JOB12345` - Batch job normal
- `STC00123` - Started Task (démon système)
- `TSU01234` - TSO user session

**Le Job ID est utilisé pour :**
- Identifier le job dans SDSF
- Voir les logs
- Purger le job
- Référencer dans les commandes

### Job Priority et CLASS

**CLASS (définie dans la JOB card) :**

```jcl
//MYJOB JOB (ACCT),'MY JOB',CLASS=A
```

**Chaque CLASS a :**
- Une priorité
- Des ressources allouées
- Un temps max d'exécution
- Des restrictions d'accès

**Exemples typiques :**
- **CLASS=A** - Production haute priorité (rapide)
- **CLASS=B** - Production normale
- **CLASS=C** - Batch overnight (lent mais gros)
- **CLASS=X** - Test/Dev (basse priorité)

**La CLASS détermine combien de temps ton job attendra.**

### MSGCLASS

**MSGCLASS contrôle où vont les messages de sortie.**

```jcl
//MYJOB JOB (ACCT),'MY JOB',CLASS=A,MSGCLASS=X
```

**Exemples :**
- **MSGCLASS=X** - Print to terminal/SDSF
- **MSGCLASS=A** - Print to printer
- **MSGCLASS=H** - Hold for review

### SDSF - System Display and Search Facility

**SDSF est L'OUTIL pour voir tes jobs dans JES2.**

**Commandes SDSF essentielles :**

**Voir tes jobs actifs :**
```
ST  (Status)
```

**Voir l'output d'un job :**
```
? jobname  (ou ? jobid)
```

**Voir tous les jobs :**
```
DA  (Display All)
```

**Purger un job :**
```
P jobid  (Purge)
```

**Hold un job :**
```
H jobid  (Hold)
```

**Release un job en hold :**
```
A jobid  (Activate)
```

### JES2 Commands

**Commandes systèmes pour contrôler JES2 :**

**Voir le statut JES2 :**
```
$DI  (Display Information)
```

**Voir les queues :**
```
$DQ  (Display Queue)
```

**Start un initiator (executeur de jobs) :**
```
$SI1  (Start Initiator 1)
```

**Stop un initiator :**
```
$PI1  (Stop Initiator 1)
```

**Drain JES2 (arrêt gracieux) :**
```
$PJ  (Pause Job Entry)
```

**Note :** Ces commandes sont réservées aux operators/sysprogs.

### JCL Control Cards JES2

**Tu peux ajouter des directives JES2 dans ton JCL.**

**Format :** `/*directive`

**Exemples :**

**1. JOBPARM - Paramètres du job**

```jcl
//MYJOB JOB ...
/*JOBPARM TIME=10  ← Max 10 minutes
```

**2. PRIORITY - Forcer la priorité**

```jcl
/*JOBPARM PRIO=15  ← Priorité 15 (1-15, 15=max)
```

**3. OUTPUT - Contrôler la sortie**

```jcl
/*OUTPUT COPIES=3  ← 3 copies du rapport
```

**4. MESSAGE - Envoyer un message**

```jcl
/*MESSAGE THIS JOB IS CRITICAL
```

### Held Output

**Tu peux "hold" l'output pour review avant impression.**

```jcl
//MYJOB JOB (ACCT),'MY JOB',CLASS=A,MSGCLASS=H
```

**Avec MSGCLASS=H, l'output reste dans JES2 jusqu'à ce que tu le release manuellement.**

**Utile pour :**
- Vérifier les résultats avant impression
- Jobs confidentiels
- Rapports à valider

### JES2 SPOOL

**Le SPOOL = Zone de stockage temporaire pour les outputs.**

**Tout ce qu'un job écrit sur SYSOUT va dans le SPOOL.**

```jcl
//SYSOUT DD SYSOUT=*  ← Va dans le SPOOL
```

**Le SPOOL est :**
- ✅ Rapide (DASD optimisé)
- ✅ Accessible par SDSF
- ❌ Temporaire (purgé après X jours)

**Si le SPOOL est plein, les jobs ne peuvent plus tourner !**

### Initiators (Exécuteurs)

**Les Initiators sont les "workers" qui exécutent les jobs.**

**Chaque initiator :**
- Peut exécuter UN job à la fois
- Est associé à certaines CLASS
- Peut avoir des restrictions (DASD, tape, etc.)

**Exemple :**
```
INI1 → Execute CLASS=A jobs
INI2 → Execute CLASS=B jobs
INI3 → Execute CLASS=A,B,C jobs
```

**Plus d'initiators = Plus de jobs en parallèle.**

**Mais :**
- CPU limité
- RAM limitée
- DASD bandwidth limité

**Balance entre throughput et performance.**

### Job Not Running ? Les Raisons Courantes

**Ton job est en INPUT mais ne tourne pas ?**

**Raisons possibles :**

1. **Pas d'initiator disponible**
   - Tous occupés par d'autres jobs
   - Attends qu'un se libère

2. **Mauvaise CLASS**
   - Aucun initiator pour cette CLASS
   - Change la CLASS

3. **Ressources manquantes**
   - Dataset en use par autre job
   - Tape drive pas disponible
   - DASD plein

4. **JOB en HOLD**
   - Quelqu'un l'a mis en hold
   - Release avec `A jobid`

5. **SPOOL plein**
   - Pas d'espace pour outputs
   - Attends le purge automatique

6. **Priorité trop basse**
   - D'autres jobs passent devant
   - Augmente la CLASS ou PRIO

### JCL pour Job Dependencies

**Comment faire tourner un job APRÈS un autre ?**

**Option 1 : Scheduler externe (Control-M, TWS)**
- Le scheduler gère les dépendances
- Méthode standard en production

**Option 2 : Started Task qui surveille**
- Un job tourne en continu
- Lance les jobs dépendants

**Option 3 : Submit depuis un job**
- Un job peut submit un autre job

```jcl
//STEP1 EXEC PGM=IEFBR14
//STEP2 EXEC PGM=INTRDR
//SUBMIT DD *
//JOB2 JOB ...
//STEP1 EXEC PGM=...
/*
```

### Cas Réel : Job Production Typique

**Exemple complet avec JES2 features :**

```jcl
//DAILYBCH JOB (PROD),'DAILY BATCH PROCESSING',
//         CLASS=A,              ← Production, haute priorité
//         MSGCLASS=H,           ← Hold output pour review
//         NOTIFY=&SYSUID,       ← Notifie l'utilisateur
//         MSGLEVEL=(1,1)        ← Messages détaillés
/*JOBPARM TIME=120              ← Max 2 heures
/*JOBPARM PRIO=10               ← Priorité élevée
//*
//* Processing step
//*
//PROCESS EXEC PGM=BATCHPGM
//SYSPRINT DD SYSOUT=*          ← Va dans le SPOOL
//INPUT    DD DSN=PROD.DAILY.TRANS,DISP=SHR
//OUTPUT   DD DSN=PROD.DAILY.REPORT(+1),
//            DISP=(NEW,CATLG),
//            UNIT=SYSDA,
//            SPACE=(CYL,(50,10))
```

### Return Codes JES2

**JES2 lui-même peut retourner des codes :**

| JCL Error | Code | Description |
|-----------|------|-------------|
| **JCL ERROR** | - | Syntax error dans le JCL |
| **SEC ERROR** | - | Security violation (RACF) |
| **ABEND** | Sxxx | System ABEND |
| **ABEND** | Uxxx | User ABEND |

**Si JCL ERROR :**
- Le job ne tourne PAS
- Fixe le JCL et resubmit

**Si SEC ERROR :**
- Pas les permissions
- Contacte RACF admin

### Monitoring JES2

**Dashboard typique d'un operator :**

```
JES2 STATUS:
  Active jobs: 45
  Queued jobs: 12
  Initiators: 8 active, 2 idle
  SPOOL usage: 65%
  
TOP JOBS:
  JOB12345 - RUNNING - CLASS=A - 15 min
  JOB12346 - RUNNING - CLASS=B - 45 min
  JOB12347 - WAITING - CLASS=A - Resources
  
ALERTS:
  SPOOL at 80% - Purge recommended
  INI3 down - Restart pending
```

### Best Practices JES2

**1. Utilise des CLASS appropriées**
```jcl
// Production urgente → CLASS=A
// Batch overnight → CLASS=C
// Test → CLASS=X
```

**2. Set des TIME limits réalistes**
```jcl
/*JOBPARM TIME=30  ← Si le job prend 25 min normalement
```

**3. NOTIFY pour jobs importants**
```jcl
// NOTIFY=&SYSUID
```

**4. MSGCLASS=H pour output sensible**
```jcl
// MSGCLASS=H  ← Review avant release
```

**5. Documente tes jobs**
```jcl
//***************************************
//* DAILY CUSTOMER PROCESSING
//* Runtime: 30-45 minutes
//* Schedule: 2:00 AM daily
//* Contact: Batch Support x1234
//***************************************
```

### Troubleshooting

**Problem : Job not starting**

1. Check SDSF status (`ST`)
2. Is it in INPUT queue ?
3. Check CLASS - any initiator for that class ?
4. Check resources - waiting for dataset ?
5. Check SPOOL - is it full ?

**Problem : Job taking too long**

1. Check what step it's on
2. Check SYSOUT for errors/warnings
3. Check if waiting for I/O
4. Contact operator if needed

**Problem : Cannot see output**

1. Job still running ? (`ST` in SDSF)
2. Job completed ? (`DA` to see all)
3. Output held ? Release with operator
4. Already purged ? Check retention

### Exercice Pratique : JES2

**Soumets un job et :**
1. Vérifie son état dans SDSF
2. Regarde l'output
3. Identifie le Job ID
4. Vérifie le Return Code

**Solution :**

```jcl
//TESTJES JOB (ACCT),'TEST JES2',CLASS=A,MSGCLASS=X
//STEP1   EXEC PGM=IEFBR14
//SYSPRINT DD SYSOUT=*
//DD1      DD DSN=TEST.FILE,
//            DISP=(NEW,CATLG),
//            UNIT=SYSDA,
//            SPACE=(TRK,(1,1))
```

**Dans SDSF :**
1. Type `ST` (Status)
2. Trouve ton job dans la liste
3. Note le Job ID (ex: JOB12345)
4. Type `?` à côté du job pour voir output
5. Vérifie le RC (Return Code) - devrait être 0

---

**📚 FIN DE LA PARTIE 3B (Chapitres 15-18)**

Tu as maintenant maîtrisé :
✅ **PROCS** - Réutiliser du JCL
✅ **Paramètres Symboliques** - Rendre les PROCS flexibles
✅ **Conditional Processing** - Contrôler l'exécution
✅ **JES2** - Comprendre le job lifecycle

**La suite (Partie 4) va couvrir :**
- Restart et Checkpoint
- Performance et Optimisation
- Sécurité et RACF
- Monitoring et Diagnostics
- Best Practices Production
- JCL en Production Réelle

**Prêt pour la Partie 4 ou tu veux digérer d'abord ?** 🚀

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

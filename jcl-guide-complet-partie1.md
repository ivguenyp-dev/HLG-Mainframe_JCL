# JCL - Job Control Language : Guide Complet Mondial
## Du Débutant Absolu au Professionnel Mainframe

**🔗 Repository GitHub :** [Learning Schooling Foundation - JCL](https://github.com/learning-schooling-foundation)

---

## 📜 NOTRE ENGAGEMENT

**Ce guide existe pour toi, où que tu sois sur cette planète.**

Tu es peut-être à Dakar, à Mumbai, à Manille, à São Paulo.  
Tu es peut-être dans un orphelinat, un camp de réfugiés, une favela.  
Tu as peut-être une connexion internet de merde, un vieux laptop, 2h d'électricité par jour.

**CE SAVOIR EST POUR TOI.**

Ce guide JCL niveau élite bancaire mondial ? Il coûte **5000€ en formation IBM**.  
Ici, il est **GRATUIT. POUR TOUJOURS.**

Tu peux le télécharger. Le copier. Le partager. Le traduire. Le modifier.  
Tu peux l'imprimer et le distribuer dans ton quartier.  
Tu peux créer une école avec.  
Tu peux changer 1000 vies avec.

**Dans 6 mois, tu peux changer ta vie.**  
**Dans 1 an, tu peux aider ta famille.**  
**Dans 2 ans, tu peux former ton quartier.**

Le JCL ne discrimine pas. Le JCL ne demande pas ton passeport.  
**Le JCL te demande juste : ES-TU PRÊT À APPRENDRE ?**

Si oui, bienvenue. Ce savoir est maintenant **LE TIEN.**

---

## Table des Matières - Partie 1

1. [Introduction au Mainframe et au JCL](#1-introduction-au-mainframe-et-au-jcl)
2. [Anatomie d'un Job JCL](#2-anatomie-dun-job-jcl)
3. [La Carte JOB - Ton Identité](#3-la-carte-job---ton-identité)
4. [La Carte EXEC - Exécuter des Programmes](#4-la-carte-exec---exécuter-des-programmes)
5. [La Carte DD - Définir les Données](#5-la-carte-dd---définir-les-données)
6. [Comprendre les Datasets](#6-comprendre-les-datasets)

---

*Note : Ce fichier contient les 6 premiers chapitres fondamentaux. Les parties suivantes couvriront : DISP et cycle de vie, utilitaires (IEBGENER, IEBCOPY, SORT, IDCAMS), procedures, conditional processing, production et best practices.*

---

## 1. Introduction au Mainframe et au JCL

### Pourquoi ce guide existe

**STOP à l'opacité IBM et aux formations hors de prix.**

Ce guide te donne les **vraies compétences** que les banques et assurances du monde entier recherchent :
- Comprendre **comment** le mainframe fonctionne (pas juste **quoi** copier-coller)
- Écrire des jobs **production-ready** dès le premier jour
- Débugger des problèmes **complexes** que tes collègues ne comprennent pas
- Gagner **45-55K€** dès ta sortie (vs 35-40K pour un dev classique)

**La différence entre un dev junior et un mainframer employable ?**

**Junior :** "J'ai copié-collé du JCL, ça marche, je sais pas pourquoi"  
**Employable :** "Je comprends exactement ce que fait chaque ligne et pourquoi"

### Qu'est-ce qu'un Mainframe ?

**Un mainframe n'est PAS un vieux dinosaure technologique.**

**Un mainframe est :**
- L'ordinateur le plus **fiable** du monde (99.999% uptime)
- L'ordinateur le plus **sécurisé** du monde
- L'ordinateur qui gère **70% des transactions bancaires mondiales**
- L'ordinateur qui fait tourner **les systèmes critiques** (banques, assurances, gouvernements, aviation)

**Exemples concrets :**
- Tu retires de l'argent au distributeur ? **Mainframe**
- Ta banque traite ton virement ? **Mainframe**
- L'assurance calcule ta prime ? **Mainframe**
- Le système de réservation Air France ? **Mainframe**
- La sécurité sociale française ? **Mainframe**

**Ces systèmes traitent :**
- 30 milliards de transactions par jour
- 87% des transactions par carte bancaire dans le monde
- 4 billions de dollars de paiements chaque jour

**Pourquoi le mainframe existe encore ?**

Parce que **rien d'autre ne peut garantir** :
1. **Fiabilité absolue** - Pas de downtime
2. **Sécurité maximale** - Données ultra-sensibles
3. **Performance extrême** - Millions de transactions/seconde
4. **Compatibilité** - Code COBOL des années 70 tourne encore

### Qu'est-ce que le JCL ?

**JCL = Job Control Language**

**Le JCL n'est PAS un langage de programmation.**

**Le JCL est un langage d'ORCHESTRATION.**

**Analogie simple :**

Imagine un chef d'orchestre :
- Le **JCL** = Le chef d'orchestre
- Les **programmes** (COBOL, SORT, etc.) = Les musiciens
- Les **datasets** (fichiers) = Les partitions

**Le chef ne joue pas de musique lui-même. Il dit aux musiciens QUOI jouer, QUAND jouer, et COMMENT jouer.**

**Le JCL fait pareil :**
```jcl
//MYJOB JOB ...           ← "Voici mon concert"
//STEP1 EXEC PGM=PROG1    ← "Musicien 1, joue ta partition"
//INPUT DD DSN=DATA.IN    ← "Voici ta partition"
//OUTPUT DD DSN=DATA.OUT  ← "Écris le résultat ici"
```

### Pourquoi le JCL est-il ESSENTIEL ?

**Sur mainframe, RIEN ne se passe sans JCL.**

Tu veux :
- Exécuter un programme COBOL ? **JCL**
- Copier un fichier ? **JCL**
- Trier des données ? **JCL**
- Faire un backup ? **JCL**
- Déployer une application ? **JCL**

**Tu ne peux PAS être mainframer sans maîtriser le JCL.**

C'est comme vouloir être chef cuisinier sans savoir utiliser un four.

### Le Marché du Travail Mainframe

**LA RÉALITÉ QUE PERSONNE NE TE DIT :**

**Pénurie MASSIVE de mainframers :**
- 50% des mainframers actuels partent à la retraite dans les 5 prochaines années
- Les universités n'enseignent PLUS le mainframe
- Les entreprises **GALÉRENT** à recruter

**Résultat pour toi :**
- Job **GARANTI** après ce guide
- Salaires **20-30% au-dessus** des dev classiques
- Sécurité d'emploi **ABSOLUE** (ces systèmes tourneront encore 30 ans)
- Demande **MONDIALE** (pas que la France)

**Salaires réels en France (2025) :**
- Junior mainframe (0-2 ans) : **42-48K€**
- Confirmé (2-5 ans) : **50-65K€**
- Senior (5+ ans) : **65-85K€**
- Expert : **85-110K€**

Compare avec un dev web junior : **35-40K€**

### Les Grandes Banques qui Recrutent

**Toutes ces banques utilisent du mainframe massivement :**

**France :**
- BNP Paribas
- Société Générale
- Crédit Agricole
- LCL (Crédit Lyonnais)
- Banque Postale
- Crédit Mutuel

**International :**
- Deutsche Bank (Allemagne)
- UBS, Credit Suisse (Suisse)
- ING (Pays-Bas)
- HSBC (UK)
- Bank of America (USA)
- Wells Fargo (USA)
- Santander (Espagne)

**Elles recrutent TOUTES en permanence.**

### Prérequis pour ce Guide

**Tu n'as besoin de RIEN savoir.**

Ce guide part de **ZÉRO ABSOLU**.

**Ce qui aide (mais n'est PAS obligatoire) :**
- Avoir déjà programmé (n'importe quel langage)
- Comprendre les concepts de fichier/dossier
- Être à l'aise avec un ordinateur

**Si tu as fait le guide COBOL de LSF, c'est PARFAIT.**  
Mais même sans, tu peux commencer ici.

### Comment Utiliser ce Guide

**1. LIS dans l'ordre**
- Chaque chapitre construit sur le précédent
- Ne saute PAS de chapitres

**2. PRATIQUE chaque exemple**
- Tape le code toi-même
- Ne copie-colle PAS
- Expérimente, change des valeurs

**3. FAIS les exercices**
- Ils sont là pour une raison
- Si tu les sautes, tu ne maîtriseras pas

**4. COMPRENDS avant d'avancer**
- Si tu ne comprends pas un concept, ARRÊTE
- Relis
- Cherche des exemples supplémentaires
- Pose des questions (GitHub issues)

**Temps estimé pour maîtriser le JCL :**
- **2-3 mois** à raison de 2h/jour
- **6 mois** à raison de 1h/jour

**C'est BEAUCOUP moins long** que d'apprendre un vrai langage de programmation.

### Ce que tu sauras après ce Guide

**Compétences techniques :**
✅ Écrire n'importe quel job JCL de zéro  
✅ Comprendre et modifier du JCL existant  
✅ Débugger des erreurs JCL complexes  
✅ Utiliser tous les utilitaires mainframe (SORT, IEBGENER, IDCAMS, etc.)  
✅ Gérer des datasets (création, copie, suppression, organisation)  
✅ Créer des procedures réutilisables  
✅ Optimiser les performances  
✅ Gérer des jobs en production  

**Compétences professionnelles :**
✅ Passer des entretiens techniques mainframe  
✅ Travailler sur du code production bancaire  
✅ Collaborer avec des équipes mainframe  
✅ Former d'autres développeurs  

**Tu pourras postuler à :**
- Développeur mainframe junior
- Analyste technique mainframe
- Ingénieur de production mainframe
- DevOps mainframe (zDevOps)

---

## 2. Anatomie d'un Job JCL

### Ton Premier Job JCL

**Avant d'apprendre les détails, regarde un job JCL complet :**

```jcl
//MYJOB01  JOB (ACCT123),'MON PREMIER JOB',
//         CLASS=A,
//         MSGCLASS=X,
//         NOTIFY=&SYSUID
//*
//* COMMENTAIRE : Ce job copie un fichier
//*
//STEP1    EXEC PGM=IEBGENER
//SYSPRINT DD SYSOUT=*
//SYSIN    DD DUMMY
//SYSUT1   DD DSN=INPUT.DATA,DISP=SHR
//SYSUT2   DD DSN=OUTPUT.DATA,
//            DISP=(NEW,CATLG,DELETE),
//            UNIT=SYSDA,
//            SPACE=(TRK,(10,5),RLSE)
```

**Ne panique PAS si tu ne comprends rien.**  
À la fin de ce chapitre, **chaque ligne sera claire**.

### Les 3 Types de Cartes (Statements)

**Un job JCL contient 3 types de cartes :**

1. **JOB** - Identifie le job
2. **EXEC** - Exécute un programme ou une procedure
3. **DD** (Data Definition) - Décrit les données

**Pense à ça comme une recette de cuisine :**

```
JOB    = Nom du plat et du chef
EXEC   = Les étapes de cuisson
DD     = Les ingrédients
```

### Structure d'une Carte JCL

**Chaque carte JCL suit ce format STRICT :**

```
//NAME     OPERATION  PARAMETERS         COMMENTS
//12345678901234567890
  ↑        ↑          ↑                  ↑
  Col 1-2  Col 3-10   Col 12-71          Col 72+
```

**RÈGLES STRICTES (le mainframe est TRÈS strict) :**

1. **Colonnes 1-2 :** Toujours `//` (sauf commentaires : `//*`)
2. **Colonnes 3-10 :** Nom (optionnel sauf pour JOB et EXEC)
3. **Colonne 11 :** ESPACE obligatoire
4. **Colonnes 12+ :** Opération et paramètres
5. **Colonne 72+ :** Commentaires (ignorés)

**⚠️ ERREUR CLASSIQUE qui va te faire perdre 2h :**

```jcl
//MYJOB JOB ...
         ↑
         Pas d'espace après le nom = ERREUR !
```

**✅ CORRECT :**

```jcl
//MYJOB    JOB ...
         ↑↑↑
         Au moins 1 espace obligatoire
```

### Les Commentaires en JCL

**3 façons de commenter :**

**1. Ligne de commentaire complète :**
```jcl
//* Ceci est un commentaire
//* Il commence par //*
```

**2. Commentaire en fin de ligne :**
```jcl
//STEP1 EXEC PGM=IEBGENER    Commentaire ici (après col 72)
```

**3. Commentaire dans les paramètres :**
```jcl
//MYJOB JOB (ACCT),    ← Paramètre comptable
//         'DESC',     ← Description
//         CLASS=A     ← Classe d'exécution
```

**💡 BONNE PRATIQUE :**

Commente **POURQUOI**, pas **QUOI** :

```jcl
//* ❌ MAUVAIS
//STEP1 EXEC PGM=IEBGENER    Exécute IEBGENER

//* ✅ BON
//STEP1 EXEC PGM=IEBGENER    Copie les transactions du jour
```

### Continuation de Ligne

**Problème :** Une ligne JCL peut devenir trop longue.

**Solution :** Continuer sur la ligne suivante.

**RÈGLES DE CONTINUATION :**

1. Arrête-toi n'importe où dans les paramètres
2. Ligne suivante commence par `//` + espace(s) + continue

**Exemple :**

```jcl
//MYJOB JOB (ACCT123),'DESCRIPTION TRÈS LONGUE QUI CONTINUE',
//         CLASS=A,
//         MSGCLASS=X,
//         NOTIFY=&SYSUID
```

**Points importants :**
- La virgule avant la continuation
- Les lignes suivantes commencent par `//` + espaces
- Indente pour la lisibilité (pas obligatoire mais recommandé)

### Les Steps (Étapes)

**Un job peut contenir plusieurs steps.**

**Pense aux steps comme des étapes de recette :**

```
Étape 1 : Préchauffer le four    → STEP1
Étape 2 : Mélanger les ingrédients → STEP2
Étape 3 : Cuire 30 minutes       → STEP3
```

**Exemple JCL avec 3 steps :**

```jcl
//MYJOB   JOB ...
//*
//STEP1   EXEC PGM=PROG1
//INPUT1  DD DSN=DATA.IN1,DISP=SHR
//OUTPUT1 DD DSN=DATA.OUT1,DISP=(NEW,CATLG)
//*
//STEP2   EXEC PGM=PROG2
//INPUT2  DD DSN=DATA.OUT1,DISP=SHR
//OUTPUT2 DD DSN=DATA.OUT2,DISP=(NEW,CATLG)
//*
//STEP3   EXEC PGM=PROG3
//INPUT3  DD DSN=DATA.OUT2,DISP=SHR
//OUTPUT3 DD DSN=DATA.FINAL,DISP=(NEW,CATLG)
```

**Qu'est-ce qui se passe ?**

1. **STEP1** lit `DATA.IN1` et crée `DATA.OUT1`
2. **STEP2** lit `DATA.OUT1` (créé par STEP1) et crée `DATA.OUT2`
3. **STEP3** lit `DATA.OUT2` (créé par STEP2) et crée `DATA.FINAL`

**C'est une chaîne de traitement (pipeline).**

### Ordre d'Exécution

**Les steps s'exécutent DANS L'ORDRE, un par un.**

```
STEP1 termine → STEP2 commence → STEP2 termine → STEP3 commence
```

**⚠️ RÈGLE CRITIQUE :**

**Si un step plante, les steps suivants NE S'EXÉCUTENT PAS** (par défaut).

```jcl
//STEP1 EXEC ...   ← S'exécute, réussit ✅
//STEP2 EXEC ...   ← S'exécute, PLANTE ❌
//STEP3 EXEC ...   ← NE S'EXÉCUTE PAS ⛔
```

*Nous verrons plus tard comment contourner ça avec le conditional processing.*

### Exemple Complet Annoté

**Reprenons notre exemple et annotant TOUT :**

```jcl
//MYJOB01  JOB (ACCT123),'MON PREMIER JOB',
//         CLASS=A,
//         MSGCLASS=X,
//         NOTIFY=&SYSUID
   ↑
   Carte JOB : Identifie le job, donne les infos comptables et d'exécution

//*
//* COMMENTAIRE : Ce job copie un fichier
//*
   ↑
   Commentaires pour expliquer ce que fait le job

//STEP1    EXEC PGM=IEBGENER
   ↑             ↑
   Nom du step   Programme à exécuter (IEBGENER = utilitaire de copie)

//SYSPRINT DD SYSOUT=*
   ↑           ↑
   Sortie des messages du programme (vers JES, affiché dans le log)

//SYSIN    DD DUMMY
   ↑           ↑
   Fichier de contrôle (ici vide car pas besoin pour une copie simple)

//SYSUT1   DD DSN=INPUT.DATA,DISP=SHR
   ↑           ↑              ↑
   Fichier     Nom du         Partagé (déjà existe, lecture seule)
   d'entrée    dataset

//SYSUT2   DD DSN=OUTPUT.DATA,
   ↑           ↑
   Fichier     Nom du dataset de sortie
   de sortie

//            DISP=(NEW,CATLG,DELETE),
               ↑    ↑      ↑
               Nouveau, si OK catalogue-le, si erreur supprime-le

//            UNIT=SYSDA,
               ↑
               Type de disque

//            SPACE=(TRK,(10,5),RLSE)
               ↑      ↑   ↑  ↑   ↑
               En tracks, 10 primaires, 5 secondaires, libère l'espace non utilisé
```

### Les Noms en JCL

**Les noms JCL suivent des règles STRICTES :**

**Règles :**
1. **1 à 8 caractères** maximum
2. Commence par une **lettre** (A-Z) ou **caractère national** (@, #, $)
3. Contient seulement : **lettres, chiffres, @, #, $**
4. **PAS d'espaces**
5. **PAS de caractères spéciaux** (-, ., /, etc.)

**✅ VALIDES :**
```jcl
//MYJOB
//STEP001
//STEP#1
//MYFILE01
//@SPECIAL
```

**❌ INVALIDES :**
```jcl
//MY-JOB      ← Contient un tiret
//VERYLONGJOBNAME  ← Plus de 8 caractères
//1STJOB      ← Commence par un chiffre
//MY JOB      ← Contient un espace
//STEP.1      ← Contient un point
```

### Conventions de Nommage

**Dans le monde professionnel, des conventions existent :**

**Pour les jobs :**
```jcl
//APPL + ENV + TYPE + SEQ
//PAYP001    ← Paie, Production, Job 001
//ACCT001    ← Accounting, Test, Job 001
```

**Pour les steps :**
```jcl
//STEP010    ← Step numéroté
//EXTRACT    ← Nom fonctionnel
//VALIDATE   ← Nom descriptif
//REPORT     ← Ce qu'il fait
```

**💡 TU créeras tes propres conventions, mais reste COHÉRENT.**

### Structure Logique d'un Job

**Pense à un job comme ça :**

```
┌─────────────────────────────────┐
│ JOB CARD                        │  ← Identification et contrôle
│ - Qui exécute le job            │
│ - Infos comptables              │
│ - Paramètres d'exécution        │
└─────────────────────────────────┘
         ↓
┌─────────────────────────────────┐
│ STEP 1                          │  ← Première étape de traitement
│ - EXEC (quel programme)         │
│ - DD statements (données)       │
└─────────────────────────────────┘
         ↓
┌─────────────────────────────────┐
│ STEP 2                          │  ← Deuxième étape
│ - EXEC (quel programme)         │
│ - DD statements (données)       │
└─────────────────────────────────┘
         ↓
┌─────────────────────────────────┐
│ STEP N                          │  ← Dernière étape
│ - EXEC (quel programme)         │
│ - DD statements (données)       │
└─────────────────────────────────┘
```

### Exercice Pratique 1

**Identifie les erreurs dans ce JCL :**

```jcl
//MY-JOB JOB (ACCT),'TEST'
//STEP1 EXEC PGM=PROG1
//INPUT DD DSN=DATA.IN DISP=SHR
//OUTPUT DD DSN=DATA.OUT,DISP=NEW
```

**Solutions :**

1. `MY-JOB` contient un tiret (invalide)
2. Pas d'espace après `DATA.IN` avant `DISP`
3. `DISP=NEW` incomplet (doit être `DISP=(NEW,CATLG)` par exemple)

**✅ CORRECT :**

```jcl
//MYJOB    JOB (ACCT),'TEST'
//STEP1    EXEC PGM=PROG1
//INPUT    DD DSN=DATA.IN,DISP=SHR
//OUTPUT   DD DSN=DATA.OUT,DISP=(NEW,CATLG)
```

---

## 3. La Carte JOB - Ton Identité

### Qu'est-ce que la Carte JOB ?

**La carte JOB est la PREMIÈRE carte de ton job.**

**Elle fait 3 choses :**
1. **Identifie** le job (nom unique)
2. **Donne des infos comptables** (qui paie pour ce job)
3. **Spécifie les paramètres d'exécution** (priorité, notifications, etc.)

**Analogie :**

Imagine que tu envoies un colis par la poste :
- **Nom** : Pour identifier ton colis
- **Comptabilité** : Qui paie l'envoi
- **Paramètres** : Urgence ? Suivi ? Notification à l'arrivée ?

**C'est exactement ce que fait la carte JOB.**

### Syntaxe de Base

```jcl
//jobname JOB (accounting-info),'description',
//            CLASS=x,
//            MSGCLASS=x,
//            MSGLEVEL=(statements,messages),
//            NOTIFY=&SYSUID
```

**Décortiquons :**

```jcl
//MYJOB01  JOB (DEPT123),'DAILY PAYROLL',CLASS=A
  ↑            ↑          ↑               ↑
  Nom du job   Info       Description     Classe d'exécution
  (1-8 car)    compta     (optionnel)     (priorité)
```

### Le Nom du Job (jobname)

**Règles pour le nom du job :**
- **1 à 8 caractères**
- Commence par **lettre** ou **@, #, $**
- Contient seulement **A-Z, 0-9, @, #, $**
- **DOIT être unique** sur le système (à cet instant)

**✅ VALIDES :**
```jcl
//PAYR001
//MONTHLY
//TEST#01
//@BACKUP
```

**❌ INVALIDES :**
```jcl
//VERYLONGJOBNAME  ← Plus de 8 caractères
//1STRUN           ← Commence par chiffre
//MY-JOB           ← Contient tiret
//PAY RUN          ← Contient espace
```

**💡 CONVENTION PROFESSIONNELLE :**

Dans les vraies entreprises, les noms suivent souvent ce pattern :

```
APPL + ENV + TYPE + SEQ
└─┬─┘ └┬┘  └─┬─┘  └┬┘
  │    │     │     │
  │    │     │     └─ Numéro séquentiel (01-99)
  │    │     └─────── Type (B=Batch, O=Online, R=Report)
  │    └─────────────  Environnement (P=Prod, T=Test, D=Dev)
  └──────────────────  Application (4 lettres)

Exemples :
PAYRPB01 = Paie, Production, Batch, Job 01
ACCTTR05 = Accounting, Test, Report, Job 05
```

### Accounting Information

**Format :**
```jcl
//MYJOB JOB (account-number)
//MYJOB JOB (account,room,programmer,project)
```

**Exemples réels :**

```jcl
//* Simple
//MYJOB JOB (DEPT123)

//* Détaillé
//MYJOB JOB (DEPT123,ROOM401,JOHN,PAYROLL)

//* Avec apostrophes si espaces
//MYJOB JOB ('DEPT 123','ROOM 401','JOHN DOE','PAYROLL PROJECT')
```

**À quoi ça sert ?**

Les entreprises **facturent** l'utilisation du mainframe en interne :
- Département de la paie utilise X heures CPU → Facturé au département
- Département comptable utilise Y heures → Facturé au département

**Pour toi, débutant :**
- Demande à ton admin système quelle valeur utiliser
- Souvent c'est juste un numéro de département ou "TEST"

**⚠️ ERREUR FRÉQUENTE :**

```jcl
//MYJOB JOB DEPT123    ← Oubli des parenthèses
```

**✅ CORRECT :**

```jcl
//MYJOB JOB (DEPT123)
```

### Programmer's Name (Description)

**C'est une chaîne de texte libre qui décrit le job.**

**Format :**
```jcl
//MYJOB JOB (acct),'description'
```

**Règles :**
- Entre **apostrophes simples**
- Maximum **20 caractères** (peut varier selon installation)
- Si contient apostrophe, double-la : `''`

**Exemples :**

```jcl
//MYJOB JOB (ACCT),'DAILY SALES REPORT'
//MYJOB JOB (ACCT),'MONTHLY BACKUP'
//MYJOB JOB (ACCT),'JOHN''S TEST JOB'
                      ↑↑
                      Apostrophe doublée
```

**💡 BONNE PRATIQUE :**

Sois **descriptif** mais **concis** :

```jcl
//* ❌ PAS CLAIR
//MYJOB JOB (ACCT),'JOB'

//* ❌ TROP LONG
//MYJOB JOB (ACCT),'THIS JOB PROCESSES THE DAILY TRANSACTION FILES'

//* ✅ PARFAIT
//MYJOB JOB (ACCT),'DAILY TXN PROCESS'
```

### CLASS Parameter

**CLASS définit la priorité et les ressources allouées à ton job.**

**Format :**
```jcl
CLASS=x
```

où `x` est une lettre (A-Z) ou un chiffre (0-9).

**Exemples :**

```jcl
//MYJOB JOB (ACCT),'PAYROLL',CLASS=A
//MYJOB JOB (ACCT),'TEST',CLASS=B
```

**Qu'est-ce que ça signifie ?**

Chaque installation mainframe définit ses propres classes :

**Exemple typique dans une banque :**

| Class | Priorité | Usage typique | CPU Max | Temps Max |
|-------|----------|---------------|---------|-----------|
| A | Haute | Production critique | Illimité | Illimité |
| B | Moyenne | Production normale | 10 min | 30 min |
| C | Basse | Tests, développement | 2 min | 10 min |
| D | Très basse | Batch de nuit | Illimité | 8h |

**⚠️ Important :**

Les classes sont **définies par ton installation**. Demande à ton admin :
- Quelle classe utiliser pour les tests ?
- Quelle classe pour la production ?

**Si tu ne spécifies pas CLASS :**

Le système utilise la classe par défaut (souvent CLASS=A).

### MSGCLASS Parameter

**MSGCLASS détermine où vont les messages de sortie (logs) du job.**

**Format :**
```jcl
MSGCLASS=x
```

**Exemples :**

```jcl
//MYJOB JOB (ACCT),'TEST',MSGCLASS=X
//MYJOB JOB (ACCT),'PROD',MSGCLASS=A
```

**Classes typiques :**

| MSGCLASS | Destination |
|----------|-------------|
| A | Imprimante physique |
| X | Spool JES (consultable en ligne) |
| H | Fichier dataset |
| T | Supprimé après consultation |

**💡 Pour toi, débutant :**

**Utilise toujours MSGCLASS=X** (ou ce que ton admin te dit).

Ça envoie les logs vers le spool JES que tu peux consulter en ligne.

### MSGLEVEL Parameter

**MSGLEVEL contrôle COMBIEN de messages tu veux voir.**

**Format :**
```jcl
MSGLEVEL=(statements,messages)
```

**Valeurs :**

**statements :**
- `0` = Affiche uniquement la carte JOB
- `1` = Affiche JOB + cartes JCL avec erreurs
- `2` = Affiche TOUT le JCL soumis

**messages :**
- `0` = Affiche seulement si le job plante
- `1` = Affiche tous les messages

**Combinaisons courantes :**

```jcl
MSGLEVEL=(1,1)  ← Standard (JCL + tous messages)
MSGLEVEL=(2,1)  ← Verbose (tout le JCL + tous messages)
MSGLEVEL=(0,0)  ← Minimal (presque rien)
```

**💡 RECOMMANDATION :**

**En apprentissage, utilise toujours :**

```jcl
MSGLEVEL=(1,1)
```

Ça te donne assez d'infos pour débugger sans te noyer dans les messages.

### NOTIFY Parameter

**NOTIFY envoie un message quand le job se termine.**

**Format :**
```jcl
NOTIFY=userid
NOTIFY=&SYSUID  ← Variable système = ton userid
```

**Exemples :**

```jcl
//MYJOB JOB (ACCT),'TEST',NOTIFY=&SYSUID
                              ↑
                              Notifie-moi quand le job termine

//MYJOB JOB (ACCT),'PROD',NOTIFY=ADMIN01
                              ↑
                              Notifie l'admin ADMIN01
```

**Qu'est-ce qui se passe ?**

Quand le job termine :
- Tu reçois un message TSO : `JOB MYJOB01 ENDED - RC=0000`
- Pratique pour les longs jobs

**&SYSUID** est une **variable système** qui contient ton userid automatiquement.

**💡 BONNE PRATIQUE :**

```jcl
NOTIFY=&SYSUID
```

Comme ça, tu es notifié automatiquement.

### TIME Parameter

**TIME limite le temps d'exécution du job.**

**Format :**
```jcl
TIME=(minutes,secondes)
TIME=1440  ← Minutes seulement
TIME=NOLIMIT  ← Pas de limite
```

**Exemples :**

```jcl
TIME=5         ← Maximum 5 minutes
TIME=(2,30)    ← Maximum 2 minutes 30 secondes
TIME=1440      ← Maximum 24 heures (1440 min)
TIME=NOLIMIT   ← Illimité
```

**Pourquoi limiter le temps ?**

**Protection contre les boucles infinies :**

```cobol
PERFORM UNTIL CONDITION
  * Si CONDITION ne devient jamais true
  * → Boucle infinie
  * → Le job tourne éternellement
  * → Bouffe les ressources du mainframe
END-PERFORM
```

**Avec TIME=5, le job s'arrête après 5 minutes même s'il boucle.**

**⚠️ Si le job dépasse TIME :**

Il est **annulé** (ABEND S322).

**💡 RECOMMANDATION :**

En test, utilise une limite raisonnable :

```jcl
TIME=10  ← 10 minutes max
```

En production, ajuste selon les besoins réels.

### REGION Parameter

**REGION définit la quantité de mémoire allouée au job.**

**Format :**
```jcl
REGION=xxxxK  ← Kilobytes
REGION=xxxxM  ← Megabytes
REGION=0M     ← Maximum disponible
```

**Exemples :**

```jcl
REGION=4096K   ← 4 MB
REGION=32M     ← 32 MB
REGION=0M      ← Maximum disponible
```

**Pourquoi spécifier REGION ?**

Si ton programme a besoin de beaucoup de mémoire (gros fichiers, tableaux, etc.), tu dois demander plus.

**⚠️ Si tu ne demandes pas assez :**

Le job plante avec **ABEND S80A** ou **S878** (mémoire insuffisante).

**💡 RECOMMANDATION :**

En test, commence avec :

```jcl
REGION=0M
```

Ça donne le maximum disponible. Ensuite, tu peux optimiser.

### TYPRUN Parameter

**TYPRUN contrôle comment le job s'exécute.**

**Valeurs :**

```jcl
TYPRUN=SCAN    ← Vérifie la syntaxe mais N'EXÉCUTE PAS
TYPRUN=HOLD    ← Met le job en attente (exécution manuelle)
TYPRUN=COPY    ← Copie le JCL vers SYSOUT
```

**TYPRUN=SCAN est SUPER UTILE pour tester :**

```jcl
//MYJOB JOB (ACCT),'TEST',TYPRUN=SCAN
```

Le système :
- ✅ Vérifie la syntaxe JCL
- ✅ Vérifie que les datasets existent
- ✅ Vérifie les autorisations
- ❌ N'EXÉCUTE PAS le job

**C'est comme faire un "dry run" ou un "plan" en Terraform.**

**💡 BONNE PRATIQUE :**

Avant de lancer un nouveau job en production :

1. Soumets avec `TYPRUN=SCAN`
2. Vérifie qu'il n'y a pas d'erreurs
3. Retire `TYPRUN=SCAN`
4. Soumets pour de vrai

### COND Parameter (Introduction)

**COND contrôle l'exécution conditionnelle du job.**

**Format simple :**
```jcl
COND=(code,operator)
```

**Exemple :**

```jcl
//MYJOB JOB (ACCT),'TEST',COND=(4,LT)
                          ↑
                          Si return code < 4, continue
                          Si return code >= 4, arrête le job
```

**Nous verrons COND en détail dans le chapitre "Conditional Processing".**

Pour l'instant, retiens juste que ça existe.

### Carte JOB Complète - Exemple Professionnel

**Voici une carte JOB typique en production bancaire :**

```jcl
//PAYRPB01 JOB (PAYROLL,RM401,JOHN,WEEKLY),
//             'WEEKLY PAYROLL RUN',
//             CLASS=A,
//             MSGCLASS=X,
//             MSGLEVEL=(1,1),
//             NOTIFY=&SYSUID,
//             TIME=30,
//             REGION=0M
//*
//* JOB: WEEKLY PAYROLL PROCESSING
//* AUTHOR: JOHN DOE
//* DATE: 2025-01-15
//* PURPOSE: CALCULATE WEEKLY SALARIES AND GENERATE REPORTS
//*
```

**Décortiquons :**

- `PAYRPB01` : Paie, Production, Batch, Job 01
- `(PAYROLL,RM401,JOHN,WEEKLY)` : Infos comptables détaillées
- `'WEEKLY PAYROLL RUN'` : Description claire
- `CLASS=A` : Production, haute priorité
- `MSGCLASS=X` : Logs vers spool en ligne
- `MSGLEVEL=(1,1)` : Tous les messages
- `NOTIFY=&SYSUID` : Notifie John à la fin
- `TIME=30` : Max 30 minutes
- `REGION=0M` : Maximum de mémoire
- Commentaires : Expliquent le contexte

**C'est le niveau de qualité attendu en production.**

### Exercice Pratique 2

**Crée une carte JOB pour ce scénario :**

Tu travailles pour une banque. Tu dois créer un job de test pour l'équipe "ACCOUNTING", qui :
- S'appelle `ACCT_TEST_01`
- Décrit "Monthly Report Test"
- Priorité basse (CLASS=C)
- Messages vers spool (MSGCLASS=X)
- Te notifie à la fin
- Limite à 5 minutes
- Utilise le maximum de mémoire

**Solution :**

```jcl
//ACCTTEST JOB (ACCOUNTING),'MONTHLY REPORT TEST',
//             CLASS=C,
//             MSGCLASS=X,
//             MSGLEVEL=(1,1),
//             NOTIFY=&SYSUID,
//             TIME=5,
//             REGION=0M
```

**Notes :**
- Nom limité à 8 caractères : `ACCTTEST` (pas `ACCT_TEST_01`)
- Tous les paramètres présents
- Commentaires ajoutés pour clarté

---

## 4. La Carte EXEC - Exécuter des Programmes

### Qu'est-ce que la Carte EXEC ?

**La carte EXEC dit au système QUEL programme exécuter.**

**Rappel de l'analogie :**
- JOB = Chef d'orchestre annonce le concert
- **EXEC = Chef dit à UN musicien de jouer**
- DD = Chef donne la partition au musicien

**Chaque STEP a UNE carte EXEC.**

### Deux Types d'EXEC

**1. EXEC PGM** - Exécute un programme
**2. EXEC PROC** - Exécute une procedure (catalogue de steps)

**On commence par EXEC PGM (le plus courant).**

### EXEC PGM - Syntaxe de Base

```jcl
//stepname EXEC PGM=program-name
```

**Exemple :**

```jcl
//STEP1 EXEC PGM=IEBGENER
  ↑          ↑
  Nom        Programme à exécuter
  du step
```

**Programmes courants :**

```jcl
//STEP1 EXEC PGM=IEBGENER   ← Utilitaire de copie
//STEP2 EXEC PGM=SORT       ← Utilitaire de tri
//STEP3 EXEC PGM=IDCAMS     ← Utilitaire VSAM
//STEP4 EXEC PGM=MYPROG     ← Ton programme COBOL
//STEP5 EXEC PGM=IEFBR14    ← Programme "do nothing" (gestion datasets)
```

### Où se Trouvent les Programmes ?

**Les programmes sont dans des bibliothèques (libraries) système.**

**Bibliothèques standard :**
- `SYS1.LINKLIB` - Programmes système
- `SYS1.LPALIB` - Programmes chargés en mémoire
- Custom libraries - Programmes de ton entreprise

**Le système cherche automatiquement dans ces bibliothèques.**

**Si ton programme est ailleurs, tu dois le spécifier avec STEPLIB** (on verra ça plus tard).

### PARM Parameter

**PARM passe des paramètres AU programme.**

**Format :**
```jcl
//STEP1 EXEC PGM=program,PARM='parameters'
```

**Exemple :**

```jcl
//STEP1 EXEC PGM=MYPROG,PARM='MODE=TEST,DEBUG=ON'
```

**Le programme MYPROG reçoit la chaîne `'MODE=TEST,DEBUG=ON'`.**

**Exemple avec SORT :**

```jcl
//STEP1 EXEC PGM=SORT,PARM='MSGPRT=CRITICAL'
                          ↑
                          Ne affiche que les messages critiques
```

**Règles PARM :**
- Maximum **100 caractères**
- Entre apostrophes si contient espaces ou caractères spéciaux
- Le programme doit être codé pour accepter ces paramètres

**💡 BONNE PRATIQUE :**

Utilise PARM pour :
- Modes d'exécution (TEST, PROD)
- Options de debug
- Paramètres variables (dates, seuils, etc.)

**Ça rend ton JCL flexible sans avoir à recompiler le programme.**

### COND Parameter (Step Level)

**COND sur la carte EXEC contrôle si CE step doit s'exécuter.**

**Format :**
```jcl
//STEP2 EXEC PGM=PROG2,COND=(code,operator,stepname)
```

**Exemple :**

```jcl
//STEP1 EXEC PGM=PROG1
//STEP2 EXEC PGM=PROG2,COND=(4,LT,STEP1)
                           ↑
                           Si return code de STEP1 < 4, N'EXÉCUTE PAS STEP2
                           Si return code de STEP1 >= 4, EXÉCUTE STEP2
```

**C'est contre-intuitif ! On verra ça en détail dans "Conditional Processing".**

### TIME Parameter (Step Level)

**TIME sur EXEC limite le temps pour CE step uniquement.**

```jcl
//STEP1 EXEC PGM=LONGPROG,TIME=(10,0)
                          ↑
                          Ce step max 10 minutes
```

**Ça override le TIME du JOB pour ce step.**

### REGION Parameter (Step Level)

**REGION sur EXEC alloue de la mémoire pour CE step.**

```jcl
//STEP1 EXEC PGM=BIGPROG,REGION=64M
                         ↑
                         64 MB pour ce step
```

**Utile si un step a besoin de plus de mémoire que les autres.**

### Programmes Utilitaires Standards

**Le mainframe fournit des utilitaires pour les tâches courantes :**

| Programme | Fonction |
|-----------|----------|
| **IEBGENER** | Copier des fichiers séquentiels |
| **IEBCOPY** | Copier/gérer des PDS (libraries) |
| **IEFBR14** | "Do nothing" (créer/supprimer datasets) |
| **SORT/DFSORT** | Trier et manipuler des données |
| **IDCAMS** | Gérer les fichiers VSAM |
| **IEBPTPCH** | Imprimer ou punch des données |
| **IEBUPDTE** | Mettre à jour des PDS |
| **ICEGENER** | Version moderne de IEBGENER |

**On verra chacun en détail dans les chapitres suivants.**

### IEFBR14 - Le Programme Spécial

**IEFBR14 est un programme qui NE FAIT RIEN.**

**Littéralement, son code assembleur est :**

```asm
BR 14    ← Branch to Register 14 (return immediately)
```

**Pourquoi c'est utile ?**

**Pour créer ou supprimer des datasets sans exécuter de programme réel.**

**Exemple - Créer un dataset :**

```jcl
//STEP1    EXEC PGM=IEFBR14
//NEWFILE  DD DSN=MY.NEW.FILE,
//            DISP=(NEW,CATLG),
//            UNIT=SYSDA,
//            SPACE=(TRK,(10,5))
```

Le programme IEFBR14 ne fait rien, mais le système :
1. Voit la carte DD avec DISP=(NEW,CATLG)
2. Crée le dataset MY.NEW.FILE
3. Le catalogue

**Exemple - Supprimer un dataset :**

```jcl
//STEP1    EXEC PGM=IEFBR14
//OLDFILE  DD DSN=MY.OLD.FILE,DISP=(OLD,DELETE)
```

IEFBR14 ne fait rien, mais le système supprime MY.OLD.FILE.

**💡 C'est LA technique standard pour gérer les datasets.**

### Exemple Complet avec Plusieurs Steps

```jcl
//MYJOB   JOB (ACCT),'MULTI-STEP JOB',CLASS=A,MSGCLASS=X
//*
//* STEP1: Create a new dataset
//*
//STEP1   EXEC PGM=IEFBR14
//NEWFILE DD DSN=TEMP.DATA,
//           DISP=(NEW,CATLG),
//           UNIT=SYSDA,
//           SPACE=(TRK,(5,2))
//*
//* STEP2: Copy data into the new dataset
//*
//STEP2   EXEC PGM=IEBGENER
//SYSPRINT DD SYSOUT=*
//SYSIN    DD DUMMY
//SYSUT1   DD DSN=SOURCE.DATA,DISP=SHR
//SYSUT2   DD DSN=TEMP.DATA,DISP=OLD
//*
//* STEP3: Sort the data
//*
//STEP3   EXEC PGM=SORT
//SYSOUT   DD SYSOUT=*
//SORTIN   DD DSN=TEMP.DATA,DISP=SHR
//SORTOUT  DD DSN=SORTED.DATA,
//            DISP=(NEW,CATLG),
//            UNIT=SYSDA,
//            SPACE=(TRK,(5,2))
//SYSIN    DD *
  SORT FIELDS=(1,10,CH,A)
/*
//*
//* STEP4: Delete temporary dataset
//*
//STEP4   EXEC PGM=IEFBR14
//TEMP    DD DSN=TEMP.DATA,DISP=(OLD,DELETE)
```

**Ce job fait :**
1. Crée un dataset temporaire
2. Copie des données dedans
3. Trie ces données vers un nouveau dataset
4. Supprime le dataset temporaire

**C'est une chaîne de traitement typique.**

### EXEC PROC - Introduction

**PROC = Procedure = Ensemble de steps pré-écrits et réutilisables.**

**Syntaxe :**

```jcl
//STEP1 EXEC PROC=procname
//STEP1 EXEC procname  ← Forme courte
```

**Exemple :**

```jcl
//STEP1 EXEC PROC=BACKUP
```

**BACKUP est une procedure qui contient plusieurs steps pour faire un backup.**

**On verra les PROC en détail dans le chapitre dédié.**

### Exercice Pratique 3

**Écris un job qui :**
1. Crée un dataset `TEST.DATA`
2. Copie `SOURCE.DATA` dans `TEST.DATA`
3. Si la copie réussit, supprime `SOURCE.DATA`

**Solution :**

```jcl
//TESTJOB  JOB (ACCT),'COPY AND DELETE',CLASS=C,MSGCLASS=X
//*
//* STEP1: Create destination dataset
//*
//STEP1    EXEC PGM=IEFBR14
//DEST     DD DSN=TEST.DATA,
//            DISP=(NEW,CATLG),
//            UNIT=SYSDA,
//            SPACE=(TRK,(10,5))
//*
//* STEP2: Copy source to destination
//*
//STEP2    EXEC PGM=IEBGENER
//SYSPRINT DD SYSOUT=*
//SYSIN    DD DUMMY
//SYSUT1   DD DSN=SOURCE.DATA,DISP=SHR
//SYSUT2   DD DSN=TEST.DATA,DISP=OLD
//*
//* STEP3: Delete source (only if STEP2 succeeded)
//*
//STEP3    EXEC PGM=IEFBR14,COND=(0,NE,STEP2)
//SOURCE   DD DSN=SOURCE.DATA,DISP=(OLD,DELETE)
```

**Explications :**
- STEP3 utilise `COND=(0,NE,STEP2)` : "Si le return code de STEP2 n'est pas 0, ne t'exécute pas"
- Donc STEP3 ne s'exécute QUE si STEP2 a réussi (RC=0)

---

## 5. La Carte DD - Définir les Données

### Qu'est-ce que la Carte DD ?

**DD = Data Definition**

**La carte DD décrit les fichiers (datasets) utilisés par un programme.**

**Analogie finale :**
- JOB = Chef annonce le concert
- EXEC = Chef dit à un musicien de jouer
- **DD = Chef donne la partition au musicien et lui dit où s'asseoir**

**Chaque fichier utilisé par un programme a UNE carte DD.**

### Structure de Base

```jcl
//ddname DD parameters
```

**Exemple :**

```jcl
//INPUT DD DSN=MY.DATA.FILE,DISP=SHR
  ↑       ↑
  DDname  Paramètres décrivant le fichier
```

### DDname - Le Nom de la DD

**Le DDname connecte la carte DD au programme.**

**Comment ça marche ?**

**Dans ton programme COBOL :**

```cobol
SELECT INPUT-FILE ASSIGN TO MYINPUT.
```

**Dans ton JCL :**

```jcl
//MYINPUT DD DSN=REAL.DATA.FILE,DISP=SHR
  ↑
  DOIT correspondre au nom dans le COBOL
```

**Le programme dit "je veux lire MYINPUT".**  
**Le JCL dit "MYINPUT pointe vers REAL.DATA.FILE".**

### DDnames Standards

**Certains programmes utilisent des DDnames standard :**

| DDname | Usage | Programme |
|--------|-------|-----------|
| **SYSPRINT** | Messages de sortie | Presque tous |
| **SYSIN** | Contrôle/commandes | Utilitaires |
| **SYSOUT** | Sortie générale | SORT, etc. |
| **SYSUT1** | Entrée | IEBGENER |
| **SYSUT2** | Sortie | IEBGENER |
| **SORTIN** | Entrée | SORT |
| **SORTOUT** | Sortie | SORT |
| **SYSDUMP** | Dump en cas d'erreur | Tous |

**Tu DOIS utiliser ces noms avec ces programmes.**

**Exemple IEBGENER :**

```jcl
//STEP1    EXEC PGM=IEBGENER
//SYSPRINT DD SYSOUT=*        ← Messages vers spool
//SYSIN    DD DUMMY           ← Pas de commandes de contrôle
//SYSUT1   DD DSN=INPUT.DATA,DISP=SHR   ← Fichier d'entrée
//SYSUT2   DD DSN=OUTPUT.DATA,DISP=(NEW,CATLG) ← Fichier de sortie
```

**IEBGENER s'attend à ces DDnames. Si tu utilises d'autres noms, ça ne marchera pas.**

### DSN Parameter - Nom du Dataset

**DSN = Data Set Name**

**Format :**

```jcl
//DD1 DD DSN=dataset-name
```

**Règles pour les noms de datasets :**

1. **Maximum 44 caractères** total
2. Composé de **qualifiers** séparés par des points
3. Chaque qualifier :
   - 1 à 8 caractères
   - Commence par lettre ou @, #, $
   - Contient lettres, chiffres, @, #, $, tiret (-)
4. **PAS d'espace**

**✅ VALIDES :**

```jcl
DSN=MY.DATA.FILE
DSN=USER.PROD.PAYROLL.MASTER
DSN=SYS1.PROCLIB
DSN=TEST.@FILE.#123
DSN=A
DSN=A.B.C.D.E.F.G.H.I.J  ← Long mais valide
```

**❌ INVALIDES :**

```jcl
DSN=MY DATA FILE        ← Espaces
DSN=MY..DATA           ← Deux points consécutifs
DSN=VERYLONGNAMEHERE   ← Qualifier > 8 caractères
DSN=MY.DATA.FILE.      ← Point final
DSN=.MY.DATA           ← Point initial
```

**Convention de Nommage Professionnelle :**

```
USER.ENV.APP.TYPE.DESC
└─┬─┘ └┬┘ └┬┘ └─┬─┘ └┬┘
  │    │   │    │    │
  │    │   │    │    └─ Description
  │    │   │    └────── Type (DATA, PGM, JCL, etc.)
  │    │   └─────────── Application
  │    └───────────────  Environnement (PROD, TEST, DEV)
  └────────────────────  Userid ou département

Exemples :
JOHN.TEST.PAYROLL.DATA.EMPLOYEES
PROD.ACCT.REPORT.JCL.MONTHLY
DEV.MY.APP.PGM.LOADLIB
```

### Temporary Datasets

**Si tu veux un dataset temporaire (supprimé à la fin du job) :**

```jcl
//TEMP DD DSN=&&TEMPFILE,DISP=(NEW,PASS)
            ↑↑
            Double esperluette = Temporaire
```

**`&&` crée un dataset temporaire unique pour CE job.**

**Exemple :**

```jcl
//STEP1   EXEC PGM=PROG1
//OUTPUT  DD DSN=&&TEMP,DISP=(NEW,PASS)

//STEP2   EXEC PGM=PROG2
//INPUT   DD DSN=&&TEMP,DISP=(OLD,DELETE)
```

STEP1 crée `&&TEMP`, STEP2 l'utilise, puis il est supprimé à la fin du job.

**💡 Pratique pour les fichiers intermédiaires dans une chaîne de traitement.**

### DISP Parameter - Disposition

**DISP est CRITIQUE. Il contrôle :**
1. Le statut AVANT l'exécution
2. Que faire si le step RÉUSSIT
3. Que faire si le step PLANTE

**Format :**

```jcl
DISP=(status,normal,abnormal)
```

**Valeurs de status :**
- `NEW` - Dataset n'existe pas, on le crée
- `OLD` - Dataset existe, accès exclusif
- `SHR` - Dataset existe, accès partagé
- `MOD` - Dataset existe, on ajoute à la fin (ou crée si n'existe pas)

**Valeurs de normal (si step réussit) :**
- `CATLG` - Catalogue le dataset
- `DELETE` - Supprime le dataset
- `KEEP` - Garde le dataset (non catalogué)
- `PASS` - Passe au step suivant (datasets temporaires)

**Valeurs de abnormal (si step plante) :**
- Mêmes valeurs que normal

**Exemples :**

```jcl
//* Lire un fichier existant
//INPUT DD DSN=EXISTING.FILE,DISP=SHR

//* Créer un nouveau fichier, le cataloguer si succès, le supprimer si erreur
//OUTPUT DD DSN=NEW.FILE,DISP=(NEW,CATLG,DELETE)

//* Modifier un fichier existant
//UPDATE DD DSN=MASTER.FILE,DISP=OLD

//* Fichier temporaire pour le step suivant
//TEMP DD DSN=&&TEMPFILE,DISP=(NEW,PASS)

//* Ajouter à un fichier
//APPEND DD DSN=LOG.FILE,DISP=MOD
```

**Forme courte (si normal et abnormal identiques) :**

```jcl
DISP=(NEW,CATLG)
     ↑     ↑
     status, normal (abnormal = même chose)
```

**Forme ultra-courte (défauts) :**

```jcl
DISP=SHR
     ↑
     = (SHR,KEEP,KEEP)
```

### DISP - Règles Critiques

**⚠️ ERREUR #1 : NEW avec fichier existant**

```jcl
//OUTPUT DD DSN=EXISTING.FILE,DISP=NEW
```

**Résultat :** JCL plante "FILE ALREADY EXISTS"

**💡 Solution :** Supprime le fichier d'abord ou utilise `DISP=OLD`

---

**⚠️ ERREUR #2 : SHR sur fichier à modifier**

```jcl
//OUTPUT DD DSN=MASTER.FILE,DISP=SHR
```

Si ton programme ÉCRIT dedans et quelqu'un d'autre le lit en même temps → **CORRUPTION DE DONNÉES**

**💡 Solution :** Utilise `DISP=OLD` pour accès exclusif

---

**⚠️ ERREUR #3 : Oublier CATLG**

```jcl
//OUTPUT DD DSN=NEW.FILE,DISP=NEW
```

Le fichier est créé mais **PAS catalogué** → Tu ne peux plus le retrouver !

**💡 Solution :**

```jcl
DISP=(NEW,CATLG)
```

---

**⚠️ ERREUR #4 : DELETE en normal disposition**

```jcl
//OUTPUT DD DSN=IMPORTANT.FILE,DISP=(NEW,DELETE)
```

Même si le step réussit, le fichier est **SUPPRIMÉ** !

**💡 Solution :** Utilise `CATLG`, pas `DELETE`

### UNIT Parameter

**UNIT spécifie le type de périphérique de stockage.**

**Valeurs courantes :**

```jcl
UNIT=SYSDA    ← Disque standard (le plus courant)
UNIT=TAPE     ← Bande magnétique
UNIT=3390     ← Type de disque spécifique
UNIT=SYSALLDA ← N'importe quel disque disponible
```

**99% du temps, tu utiliseras :**

```jcl
UNIT=SYSDA
```

**Exemple :**

```jcl
//OUTPUT DD DSN=NEW.FILE,
//          DISP=(NEW,CATLG),
//          UNIT=SYSDA,
//          SPACE=(TRK,(10,5))
```

### SPACE Parameter

**SPACE alloue de l'espace disque pour un nouveau dataset.**

**Format de base :**

```jcl
SPACE=(unit,(primary,secondary))
```

**Units disponibles :**
- `TRK` - Tracks (pistes)
- `CYL` - Cylinders (cylindres)
- `nnnnn` - Nombre de bytes par bloc

**Exemple :**

```jcl
SPACE=(TRK,(10,5))
       ↑    ↑   ↑
       │    │   └─ Allocation secondaire : 5 tracks à la fois si besoin
       │    └───── Allocation primaire : 10 tracks
       └────────── Unité : tracks
```

**Comment ça marche ?**

1. Le système alloue **10 tracks** initialement
2. Si le fichier grandit et remplit ces 10 tracks, le système alloue automatiquement **5 tracks supplémentaires**
3. Si encore plein, alloue encore **5 tracks**
4. Et ainsi de suite jusqu'à **15 extensions** maximum

**Quelle taille choisir ?**

**Règle approximative :**
- 1 track ≈ 56 KB (sur disque 3390)
- 1 cylinder = 15 tracks ≈ 840 KB

**Exemples pratiques :**

```jcl
//* Petit fichier (quelques KB)
SPACE=(TRK,(1,1))

//* Fichier moyen (quelques MB)
SPACE=(TRK,(50,10))

//* Gros fichier (dizaines de MB)
SPACE=(CYL,(10,2))

//* Très gros fichier (centaines de MB)
SPACE=(CYL,(100,20))
```

**💡 BONNE PRATIQUE :**

Commence avec une estimation raisonnable. Si le fichier est trop gros, le job plantera avec "NO SPACE" et tu pourras ajuster.

### SPACE - Options Avancées

**RLSE (Release) :**

```jcl
SPACE=(TRK,(100,10),RLSE)
                     ↑
                     Libère l'espace non utilisé à la fin
```

Si tu alloues 100 tracks mais utilises seulement 30, `RLSE` rend les 70 tracks inutilisés au système.

**💡 Utilise toujours RLSE pour économiser l'espace disque.**

---

**CONTIG (Contiguous) :**

```jcl
SPACE=(CYL,(10,5),RLSE,CONTIG)
                        ↑
                        Espace contigu requis
```

Force le système à allouer l'espace de manière contiguë (performance).

---

**Directory Blocks (pour PDS seulement) :**

```jcl
SPACE=(TRK,(10,5,10),RLSE)
                  ↑
                  10 directory blocks
```

On verra ça dans le chapitre sur les PDS.

### SYSOUT Parameter

**SYSOUT envoie la sortie vers le spool JES** (pour consultation ou impression).

**Format :**

```jcl
//ddname DD SYSOUT=class
```

**Exemples :**

```jcl
//SYSPRINT DD SYSOUT=*
                    ↑
                    * = même classe que MSGCLASS du JOB

//REPORT DD SYSOUT=A
                  ↑
                  Classe A (souvent = imprimante)

//LOG DD SYSOUT=X
              ↑
              Classe X (spool en ligne)
```

**💡 Pour les messages et logs, utilise toujours :**

```jcl
//SYSPRINT DD SYSOUT=*
```

### DUMMY Parameter

**DUMMY crée un fichier "vide" (ignoré).**

```jcl
//ddname DD DUMMY
```

**Utilisé quand un programme attend un fichier mais tu n'en as pas besoin.**

**Exemple avec IEBGENER :**

```jcl
//STEP1    EXEC PGM=IEBGENER
//SYSPRINT DD SYSOUT=*
//SYSIN    DD DUMMY          ← Pas de commandes de contrôle
//SYSUT1   DD DSN=INPUT,DISP=SHR
//SYSUT2   DD DSN=OUTPUT,DISP=(NEW,CATLG)
```

IEBGENER attend `SYSIN` mais on n'a pas de commandes, donc `DUMMY`.

### Instream Data (DD *)

**Tu peux mettre des données directement dans le JCL :**

```jcl
//ddname DD *
data ligne 1
data ligne 2
data ligne 3
/*
```

**Le `/*` marque la fin des données.**

**Exemple avec SORT :**

```jcl
//STEP1   EXEC PGM=SORT
//SYSIN   DD *
  SORT FIELDS=(1,10,CH,A)
/*
```

Les lignes entre `DD *` et `/*` sont les **commandes de contrôle pour SORT**.

**⚠️ ATTENTION :**

Les données instream sont **limitées à 80 caractères par ligne** (format carte).

### Concatenation de Datasets

**Tu peux lire plusieurs datasets comme un seul :**

```jcl
//INPUT   DD DSN=FILE1,DISP=SHR
//        DD DSN=FILE2,DISP=SHR
//        DD DSN=FILE3,DISP=SHR
```

Le programme lit FILE1, puis FILE2, puis FILE3 **comme s'ils étaient un seul fichier**.

**Règles :**
1. Les DDnames suivants sont **vides** (pas de DDname)
2. Les datasets doivent avoir des **caractéristiques compatibles** (même format)

**💡 Utile pour combiner des fichiers de données.**

### Exemple Complet de Step avec DD Statements

```jcl
//COPYSTEP EXEC PGM=IEBGENER
//*
//* Messages de sortie vers le spool
//SYSPRINT DD SYSOUT=*
//*
//* Pas de commandes de contrôle
//SYSIN    DD DUMMY
//*
//* Fichier d'entrée (existant, partagé)
//SYSUT1   DD DSN=PROD.MASTER.DATA,
//            DISP=SHR
//*
//* Fichier de sortie (nouveau, catalogué si succès)
//SYSUT2   DD DSN=BACKUP.MASTER.DATA,
//            DISP=(NEW,CATLG,DELETE),
//            UNIT=SYSDA,
//            SPACE=(CYL,(10,2),RLSE),
//            DCB=(RECFM=FB,LRECL=80,BLKSIZE=8000)
```

*Note : DCB sera expliqué dans le prochain chapitre.*

### Exercice Pratique 4

**Écris un step qui :**
1. Copie `SOURCE.DATA` vers `DEST.DATA`
2. Si `DEST.DATA` existe déjà, le remplace
3. Alloue 50 tracks primaires, 10 secondaires
4. Libère l'espace non utilisé

**Solution :**

```jcl
//STEP1    EXEC PGM=IEFBR14
//OLD      DD DSN=DEST.DATA,DISP=(MOD,DELETE)  ← Supprime si existe

//STEP2    EXEC PGM=IEBGENER
//SYSPRINT DD SYSOUT=*
//SYSIN    DD DUMMY
//SYSUT1   DD DSN=SOURCE.DATA,DISP=SHR
//SYSUT2   DD DSN=DEST.DATA,
//            DISP=(NEW,CATLG),
//            UNIT=SYSDA,
//            SPACE=(TRK,(50,10),RLSE)
```

---

## 6. Comprendre les Datasets

### Qu'est-ce qu'un Dataset ?

**Sur mainframe, on ne dit PAS "fichier" (file).**  
**On dit "dataset".**

**Un dataset est l'unité de stockage de données sur mainframe.**

**Équivalences approximatives :**

| Mainframe | Windows/Linux |
|-----------|---------------|
| Dataset | Fichier |
| PDS (Partitioned Dataset) | Dossier/Répertoire |
| Member (d'un PDS) | Fichier dans un dossier |
| Volume | Disque dur |
| Catalogue | Index des fichiers |

### Types de Datasets

**3 types principaux :**

1. **Sequential (PS)** - Fichier séquentiel simple
2. **Partitioned (PDS/PDSE)** - "Dossier" contenant des membres
3. **VSAM** - Virtual Storage Access Method (avancé)

**On commence par les Sequential.**

### Sequential Datasets (PS)

**PS = Physical Sequential**

**C'est le type le plus simple : une suite d'enregistrements.**

```
Record 1
Record 2
Record 3
Record 4
...
```

**Tu lis séquentiellement du début à la fin.**

**Analogie :** Une cassette VHS. Tu dois dérouler pour lire.

**Exemple de création :**

```jcl
//STEP1   EXEC PGM=IEFBR14
//MYFILE  DD DSN=MY.SEQUENTIAL.FILE,
//           DISP=(NEW,CATLG),
//           UNIT=SYSDA,
//           SPACE=(TRK,(10,5),RLSE),
//           DCB=(RECFM=FB,LRECL=80,BLKSIZE=8000)
```

### DCB - Data Control Block

**DCB décrit la STRUCTURE des données dans le dataset.**

**3 paramètres essentiels :**

1. **RECFM** - Record Format
2. **LRECL** - Logical Record Length
3. **BLKSIZE** - Block Size

### RECFM - Record Format

**RECFM définit comment les enregistrements sont organisés.**

**Valeurs courantes :**

| RECFM | Signification | Usage |
|-------|---------------|-------|
| **F** | Fixed | Enregistrements longueur fixe, 1 par bloc |
| **FB** | Fixed Blocked | Longueur fixe, plusieurs par bloc |
| **V** | Variable | Enregistrements longueur variable |
| **VB** | Variable Blocked | Longueur variable, plusieurs par bloc |
| **U** | Undefined | Pas de structure (rare) |

**Le plus courant : FB (Fixed Blocked)**

### LRECL - Logical Record Length

**LRECL = La longueur d'UN enregistrement en bytes.**

**Exemples :**

```jcl
LRECL=80   ← Chaque enregistrement fait 80 bytes (format carte)
LRECL=100  ← Chaque enregistrement fait 100 bytes
LRECL=500  ← Chaque enregistrement fait 500 bytes
```

**Pour RECFM=FB, tous les enregistrements font EXACTEMENT cette taille.**

### BLKSIZE - Block Size

**BLKSIZE = Combien de bytes lus/écrits à la fois.**

**Pour RECFM=FB :**

```
BLKSIZE = LRECL × nombre d'enregistrements par bloc
```

**Exemple :**

```jcl
LRECL=80
BLKSIZE=8000
```

8000 / 80 = 100 enregistrements par bloc.

**Pourquoi c'est important ?**

**Performance.** Plus le BLKSIZE est grand, moins d'I/O (entrées/sorties).

**Règle :**

**BLKSIZE doit être un multiple de LRECL.**

```jcl
LRECL=80, BLKSIZE=8000  ← ✅ 8000 / 80 = 100 (OK)
LRECL=80, BLKSIZE=8001  ← ❌ 8001 / 80 = 100.0125 (PAS OK)
```

### DCB Complet - Exemple

```jcl
//MYFILE DD DSN=DATA.FILE,
//          DISP=(NEW,CATLG),
//          UNIT=SYSDA,
//          SPACE=(TRK,(10,5),RLSE),
//          DCB=(RECFM=FB,LRECL=80,BLKSIZE=27920)
             ↑    ↑       ↑          ↑
             │    │       │          └─ Taille du bloc (optimal pour 3390)
             │    │       └──────────── Longueur d'un enregistrement
             │    └──────────────────── Format : Fixed Blocked
             └───────────────────────── Data Control Block
```

**💡 BLKSIZE optimal pour disque 3390 :**

Le plus efficace est un **demi-track** = **27920 bytes**.

Donc si tu ne sais pas quoi mettre :

```jcl
BLKSIZE=27920
```

Ajuste LRECL pour que 27920 soit un multiple.

**Exemples :**
- LRECL=80, BLKSIZE=27920 → 27920 / 80 = 349 enregistrements/bloc ✅
- LRECL=100, BLKSIZE=27900 → 27900 / 100 = 279 enregistrements/bloc ✅

### Catalogues

**Le CATALOGUE est l'index des datasets.**

**Quand tu catalogues un dataset (DISP=CATLG), le système enregistre :**
- Nom du dataset
- Volume (disque) où il se trouve
- Caractéristiques (RECFM, LRECL, etc.)

**Pourquoi cataloguer ?**

**Pour retrouver le dataset facilement :**

```jcl
//INPUT DD DSN=MY.DATA.FILE,DISP=SHR
```

Le système consulte le catalogue, trouve que `MY.DATA.FILE` est sur le volume `WORK01`, et l'ouvre.

**Si le dataset N'EST PAS catalogué, tu dois spécifier le volume :**

```jcl
//INPUT DD DSN=MY.DATA.FILE,DISP=SHR,VOL=SER=WORK01
```

**💡 TOUJOURS catalogue tes datasets importants.**

### Supprimer un Dataset

**3 façons de supprimer un dataset :**

**1. IEFBR14 avec DISP=DELETE :**

```jcl
//STEP1 EXEC PGM=IEFBR14
//DEL   DD DSN=OLD.FILE,DISP=(OLD,DELETE)
```

**2. IDCAMS DELETE :**

```jcl
//STEP1  EXEC PGM=IDCAMS
//SYSPRINT DD SYSOUT=*
//SYSIN    DD *
  DELETE OLD.FILE
/*
```

**3. TSO DELETE :**

En ligne de commande TSO : `DELETE 'OLD.FILE'`

### Renommer un Dataset

**Utilise IEFBR14 avec DISP=OLD et nouveau nom :**

Non, en fait, **tu ne peux PAS renommer directement avec JCL.**

**Solution : Copie puis supprime l'ancien.**

```jcl
//STEP1   EXEC PGM=IEBGENER
//SYSPRINT DD SYSOUT=*
//SYSIN    DD DUMMY
//SYSUT1   DD DSN=OLD.NAME,DISP=SHR
//SYSUT2   DD DSN=NEW.NAME,DISP=(NEW,CATLG)

//STEP2   EXEC PGM=IEFBR14
//DEL     DD DSN=OLD.NAME,DISP=(OLD,DELETE)
```

**Ou utilise IDCAMS ALTER :**

```jcl
//STEP1  EXEC PGM=IDCAMS
//SYSPRINT DD SYSOUT=*
//SYSIN    DD *
  ALTER OLD.NAME NEWNAME(NEW.NAME)
/*
```

### Exemple Complet - Gestion de Datasets

```jcl
//DSMGMT  JOB (ACCT),'DATASET MANAGEMENT',CLASS=A,MSGCLASS=X
//*
//* STEP1: Create a new sequential dataset
//*
//STEP1   EXEC PGM=IEFBR14
//NEWFILE DD DSN=PROD.MASTER.DATA,
//           DISP=(NEW,CATLG),
//           UNIT=SYSDA,
//           SPACE=(CYL,(5,1),RLSE),
//           DCB=(RECFM=FB,LRECL=100,BLKSIZE=27900)
//*
//* STEP2: Copy data into the new dataset
//*
//STEP2   EXEC PGM=IEBGENER
//SYSPRINT DD SYSOUT=*
//SYSIN    DD DUMMY
//SYSUT1   DD DSN=TEST.SAMPLE.DATA,DISP=SHR
//SYSUT2   DD DSN=PROD.MASTER.DATA,DISP=OLD
//*
//* STEP3: Create a backup
//*
//STEP3   EXEC PGM=IEBGENER
//SYSPRINT DD SYSOUT=*
//SYSIN    DD DUMMY
//SYSUT1   DD DSN=PROD.MASTER.DATA,DISP=SHR
//SYSUT2   DD DSN=BACKUP.MASTER.DATA,
//           DISP=(NEW,CATLG),
//           UNIT=SYSDA,
//           SPACE=(CYL,(5,1),RLSE),
//           DCB=(RECFM=FB,LRECL=100,BLKSIZE=27900)
//*
//* STEP4: Delete the test dataset
//*
//STEP4   EXEC PGM=IEFBR14
//DEL     DD DSN=TEST.SAMPLE.DATA,DISP=(OLD,DELETE)
```

**Ce job :**
1. Crée un dataset de production
2. Y copie des données de test
3. Crée un backup
4. Supprime le dataset de test

### Exercice Pratique 5

**Crée un job qui :**
1. Crée un dataset `MY.NEW.FILE` avec :
   - Format FB
   - LRECL=80
   - BLKSIZE optimal (27920)
   - 20 tracks primaires, 5 secondaires
   - Libère l'espace non utilisé
2. Copie `SAMPLE.DATA` dedans
3. Crée un deuxième dataset `MY.BACKUP.FILE` avec les mêmes caractéristiques
4. Copie `MY.NEW.FILE` dans `MY.BACKUP.FILE`

**Solution :**

```jcl
//EXERCISE JOB (ACCT),'EXERCISE 5',CLASS=C,MSGCLASS=X
//*
//* STEP1: Create first dataset
//*
//STEP1   EXEC PGM=IEFBR14
//FILE1   DD DSN=MY.NEW.FILE,
//           DISP=(NEW,CATLG),
//           UNIT=SYSDA,
//           SPACE=(TRK,(20,5),RLSE),
//           DCB=(RECFM=FB,LRECL=80,BLKSIZE=27920)
//*
//* STEP2: Copy sample data into it
//*
//STEP2   EXEC PGM=IEBGENER
//SYSPRINT DD SYSOUT=*
//SYSIN    DD DUMMY
//SYSUT1   DD DSN=SAMPLE.DATA,DISP=SHR
//SYSUT2   DD DSN=MY.NEW.FILE,DISP=OLD
//*
//* STEP3: Create backup dataset
//*
//STEP3   EXEC PGM=IEFBR14
//FILE2   DD DSN=MY.BACKUP.FILE,
//           DISP=(NEW,CATLG),
//           UNIT=SYSDA,
//           SPACE=(TRK,(20,5),RLSE),
//           DCB=(RECFM=FB,LRECL=80,BLKSIZE=27920)
//*
//* STEP4: Copy to backup
//*
//STEP4   EXEC PGM=IEBGENER
//SYSPRINT DD SYSOUT=*
//SYSIN    DD DUMMY
//SYSUT1   DD DSN=MY.NEW.FILE,DISP=SHR
//SYSUT2   DD DSN=MY.BACKUP.FILE,DISP=OLD
```

---

## Conclusion de la Partie 1

**🎉 BRAVO ! Tu as complété les 6 premiers chapitres fondamentaux du JCL !**

### Ce que tu maîtrises maintenant

**Concepts Fondamentaux :**
✅ **Mainframe et JCL** - Comprendre l'écosystème mainframe et le rôle du JCL  
✅ **Anatomie d'un Job** - Structure JOB/EXEC/DD, syntaxe, commentaires  
✅ **Carte JOB** - Identification, accounting, CLASS, MSGCLASS, NOTIFY, etc.  
✅ **Carte EXEC** - Exécuter des programmes, PARM, utilitaires standards  
✅ **Carte DD** - Définir des données, DSN, DISP, UNIT, SPACE, SYSOUT  
✅ **Datasets** - Types, DCB, RECFM, LRECL, BLKSIZE, catalogues  

### Tu es capable de :

🚀 **Lire et comprendre** n'importe quel job JCL basique  
🚀 **Écrire** des jobs JCL simples de zéro  
🚀 **Créer, copier, supprimer** des datasets  
🚀 **Utiliser** IEFBR14 et IEBGENER  
🚀 **Comprendre** les messages d'erreur JCL  
🚀 **Débugger** des problèmes JCL courants  

### Prochaines Étapes : Partie 2

**La Partie 2 couvrira les chapitres 7-12 :**

7. **DISP - Gérer le Cycle de Vie** - Maîtriser DISP en profondeur  
8. **Organisation des Datasets** - Sequential, PDS, PDSE, VSAM  
9. **GDG - Generation Data Groups** - Versioning de datasets  
10. **Catalogues et Volumes** - Gestion avancée  
11. **IEBGENER - Copier des Données** - Maîtrise complète  
12. **IEBCOPY - Gérer les PDS** - Copie et maintenance de libraries  

**Les Parties suivantes couvriront :**
- Utilitaires avancés (SORT, IDCAMS)
- Procedures (PROCS) et paramètres symboliques
- Conditional processing
- Production et best practices
- JCL en environnement DevOps moderne

### Conseils pour Continuer

**1. PRATIQUE** 
Si tu as accès à un mainframe (via université, employeur, ou émulateur), **pratique** chaque exemple.

**2. EXPÉRIMENTE**
Change les paramètres, casse les choses, comprends les erreurs.

**3. LIS DU VRAI JCL**
Si tu travailles déjà avec du mainframe, lis le JCL de production et essaie de tout comprendre.

**4. CONSTRUIS UN PROJET**
Crée une petite application :
- Fichier d'employés
- Programme de calcul de paie (COBOL)
- Jobs JCL pour orchestrer le tout

**5. REJOINS LA COMMUNAUTÉ**
- GitHub : Contribue, pose des questions
- Forums mainframe
- Groupes LinkedIn mainframe

### Resources Additionnelles

**Émulateurs Mainframe Gratuits :**
- **Hercules** - Émulateur z/OS open source
- **zPDT** - IBM Personal Developer Tool (payant mais abordable pour étudiants)
- **Master the Mainframe** - Concours IBM avec accès gratuit

**Documentation IBM :**
- JCL Reference
- JCL User's Guide
- z/OS MVS Utilities

**Mais souviens-toi : CE GUIDE EST TON MEILLEUR ALLIÉ.**

Il est fait pour toi, avec pédagogie, clarté, et des exemples réels.

---

**Tu es maintenant sur la voie pour devenir un mainframer professionnel capable de gagner 45-55K€ dès la sortie !** 💪🔥

**Continue vers la Partie 2 pour maîtriser les concepts avancés !**

---

**📚 JCL - Job Control Language : Guide Complet Mondial - Partie 1**  
**💎 100% Gratuit • Pour Tous • À Jamais**  
**🔗 GitHub : Learning Schooling Foundation**  
**🌍 Pour les kids de Dakar, Mumbai, São Paulo, et partout dans le monde**

**En mémoire d'Aaron Swartz (1986-2013)**  
*"Information is power. But like all power, there are those who want to keep it for themselves."*

---

# TP4

Vous trouverez dans ce [lien](https://docs.google.com/presentation/d/1tonsLynHdpmkTqB0wCP2WNfBjltlScxK00hekP2wruo/edit?usp=sharing) la présentation utilisée dans ce TP.

## Introduction

Dans cette partie, nous allons détailler le langage de contrôle des données.

## Gestion des privilèges

Un privilège donne le droit d'exécuter certaines commandes SQL ou le droit d'accéder à certaines ressources.

Oracle possède deux types de privilèges :
- les privilèges systèmes
- les privilèges objets.

Un privilège peut être affecté (retiré) à:
- un Utilisateur,
- un Rôle
- tous les utilisateurs (PUBLIC).

Les privilèges donnent le droit de réaliser des opérations systèmes.

Ces privilèges sont classés par catégories d'objets.

Exemple de privilèges systèmes de la catégorie TABLE:
- CREATE TABLE 
- CREATE ANY TABLE
- ALTER ANY TABLE 
- BACKUP ANY TABLE
- DROP ANY TABLE 
- SELECT ANY TABLE
- INSERT ANY TABLE 
- UPDATE ANY TABLE
- DELETE ANY TABLE 
- ...

Affectation d'un privilège : 
```GRANT { system_priv | role } TO { user | role | PUBLIC }```

Révocation d’un privilège :
```REVOKE { <system_priv> | <rôle> } FROM { <utilisateur> | <rôle> | PUBLIC }```

## Gestion des rôles

Un rôle est un concept Oracle qui permet de regrouper plusieurs privilèges et /ou rôles afin de les affecter ou retirer en bloc à un utilisateur et /ou un rôle.

Un rôle facilite la gestion des privilèges.

Pour créer un rôle, il faut avoir le privilège ```CREATE ROLE```

<p align="center">
  <img width="750" src="images/1.png" alt="picture">
</p>

<p align="center">
  <img width="750" src="images/2.png" alt="picture">
</p>

Pour créer un rôle : ```CREATE ROLE rôle [ { NOT IDENTIFIED | IDENTIFIED { BY password | EXTERNALLY | GLOBALLY | USING package} ]```
avec : 
- NOT IDENTIFIED : permet de créer un rôle sans mot de passe
- EXTERNALLY : mot de passe est contrôlé au niveau de l'OS
- GLOBALLY : Rôle autorisé au niveau de l’annuaire
- USING package : rôle applicatif.

Pour supprimer un rôle : ```DROP ROLE role```

## Gestion des profils

Un profile est un concept Oracle qui permet à l'administrateur d'une base de contrôler la consommation des ressources systèmes et des mots de passes.

Pour créer un profil : ``` CREATE PROFILE profile LIMIT[ SESSIONS_PER_USER { integer | UNLIMITED | DEFAULT} ][ CPU_PER_SESSION { integer | UNLIMITED | DEFAULT } ][ CPU_PER_CALL { integer | UNLIMITED | DEFAULT } ][ CONNECT_TIME { integer | UNLIMITED | DEFAULT } ][ IDLE_TIME { integer | UNLIMITED | DEFAULT } ][LOGICAL_READS_PER_SESSION {integer | UNLIMITED|DEFAULT}][LOGICAL_READS_PER_CALL {integer | UNLIMITED|DEFAULT}][ COMPOSITE_LIMIT { integer | UNLIMITED | DEFAULT } ][PRIVATE_SGA {integer [K | M] | UNLIMITED | DEFAULT}]```
avec : 
- Session_per_user : Nombre maximum de sessions par utilisateur
- Logical_read_per_session : Nbre de blocs de données à lire pour une session
- cpu_per_session : temps CPU max par session en % de sécondes
- cpu_per_call : temps CPU pour un appel (en cas de parse, execute ou fetch) en % de secondes
- connect_time : temps écoulé maximum (en minutes)
- idle_time : temps maximum d'inactivité.
- private_sga : taille privée de la SGA allouée à un utilisateur
- unlimited : limite de la ressource illimitée
- default : prend la limite par défaut de la ressource.

## Gestion des utilisateurs
Lors de la création d'un utilisateur, il est possible de lui affecter : un mot de passe, un tablespace par défaut, un tablespace temporaire, un profile (explicite ou implicite), des quotas sur les tablespaces.

```CREATE USER user IDENTIFIED { BY password | EXTERNALLY| GLOBALLY AS‘nom_externe’ } [ DEFAULT TABLESPACE tablespace ] [ TEMPORARY TABLESPACEtablespace ] [ QU0TA { integer [ K | M ] | UNLIMITED } ON tablespace ] ...[ PROFILE profile ] [PASSWORD EXPIRE] [ACCOUNT {LOCK | UNLOCK}]```
avec : 
- Externally : utilisateur authentifié par l'OS
- globally as : accès autorisé par l’annuaire LDAP

Lors de la modification d'un utilisateur, il est possible de lui affecter : un mot de passe, un tablespace par défaut, un tablespace temporaire, un profile (explicite ou implicite), des quotas sur les tablespaces et un rôle par défaut (parmi les rôles attribués par la commande grant).

 Exemples : 
 ```ALTER USER etu1_21 IDENTIFIED BY md2000p DEFAULT TABLESPACE usersQUOTA UNLIMITED ON users QUOTA 1M ON system;```
``` ALTER USER etu1_21 DEFAULT ROLE role_etu1;```

 La suppression d'un utilisateur entraîne la suppression des objets de son schéma (tables, vues, séquences, synonymes, indexes ... ) . Le privilège drop user est requis.
 
 ```DROP USER ETU1_33;``` -> suppression d'un utilisateur lié à un schéma vide
 
 ```DROP USER ETU1_33 CASCADE; ```
 
 - CASCADE : supprime  les objets du schéma de l'utilisateur et les contraintes d'intégrité de référence (et vide la corbeille).
 Rq : les rôles créés par l'utilisateur ne sont pas supprimés.
 
**La portée des privilèges :** 

``` _________________________________________________
Object privilege | Table   | View    | Sequence  |
-------------------------------------------------
ALTER            |   Yup   |   Nope  |  Yup      |
-------------------------------------------------
DELETE           |   Yup   |   Yup   |  Nope     |
-------------------------------------------------
INDEX            |   Yup   |   Nope  |  Nope     |
-------------------------------------------------
INSERT           |   Yup   |   Yup   |  Nope     |
-------------------------------------------------
SELECT           |   Yup   |   Yup   |  Yup      |
-------------------------------------------------
UPDATE           |   Yup   |   Yup   |  Nope     |
_________________________________________________
```


**Pour confirmer/lister les privilèges attribuées sur chaque Type d'Objet par rôle :**
```
__________________________________________________________________________________________________________________________
Vue de privs dans le dictionnaire de données Oracle |     Usage/Displayed Data
---------------------------------------------------------------------------------------------------------------------------
SELECT * From ROLE_SYS_PRIVS  ;                     | -->  System privileges granted to roles                              |
---------------------------------------------------------------------------------------------------------------------------
SELECT * From ROLE_TAB_PRIVS  ;                     | -->  Table privileges granted to roles                               |
---------------------------------------------------------------------------------------------------------------------------
SELECT * From USER_ROLE_PRIVS ;                     | -->  Roles accessible by the user	                                   |
---------------------------------------------------------------------------------------------------------------------------
SELECT * From USER_SYS_PRIVS  ;                     | -->  System privileges granted to the user                           |
---------------------------------------------------------------------------------------------------------------------------
SELECT * From USER_TAB_PRIVS_MADE ;                 | -->  Object privileges granted on the user’s objects                 |
---------------------------------------------------------------------------------------------------------------------------
SELECT * From USER_TAB_PRIVS_RECD ;                 | -->  Object privileges granted to the user    	                   |
---------------------------------------------------------------------------------------------------------------------------
SELECT * From USER_COL_PRIVS_MADE ;                 | -->  Object privileges granted on the columns of the user’s objects  |
---------------------------------------------------------------------------------------------------------------------------
SELECT * From USER_COL_PRIVS_RECD ;                 | -->  Object privileges granted to the user on specific columns       |
___________________________________________________________________________________________________________________________

```

 
 ## DEMO 

 - **Créer les nouveaux utilisateurs comme suit:**  
      A) Equipe Dev :

      * [ username: dev1, password: dev1 ]
      * [ username: dev2, password: dev2 ]
      
      B) Equipe Test :

      * [ username: tester1, password: tester1 ]
      * [ username: tester2, password: tester2 ]
     
     
     C) Equipe DevSecOps :      
      * [ username: devsecops1, password: devsecops1 ]
      * [ username: devsecops2, password: devsecops2 ]


```sql
create user dev1 identified by dev1;
create user dev2 identified by dev2;
create user tester1 identified by tester1;
create user tester2 identified by tester2;
create user devsecops1 identified by devsecops1;
create user devsecops2 identified by devsecops1;
result in sqlPLUS :
SQL> select* from all_users order by created desc;

USERNAME                          USER_ID CREATED
------------------------------ ---------- --------
DEV2                                   63 12/05/21
DEVSECOPS2                             62 12/05/21
TESTER2                                60 12/05/21
TESTER1                                59 12/05/21
DEV1                                   58 12/05/21

```
  --->  **Une fois qu'un utilisateur est créé, le DBA peut octroyer des privilèges de système spécifiques à cet utilisateur.**
 

  - **Attribuer les privilèges ci-dessous à l'utilisateur dev1 :** 
 
     * Création de procédures stockées.
     * Création de vue.
     * Création de séquence.
     * Création de session.
     * Création,lecture, modification de structure et suppression de tables.

```sql
SQL> grant CREATE PROCEDURE , CREATE SEQUENCE , CREATE ANY TABLE ,SELECT ANY TABLE, DROP ANY TABLE,ALTER ANY TABLE, CREATE VIEW , CREATE SESSION to dev1;

Grant succeeded.
```

¤   **Une fois qu'un utilisateur est créé, le DBA peut octroyer des privilèges de système spécifiques à cet utilisateur.**
 
 
   - **Révoquer tous les privilèges associès à l'utilisateur dev1 :** 

```sql
SQL> REVOKE  CREATE PROCEDURE , CREATE SEQUENCE , CREATE ANY TABLE ,SELECT ANY TABLE, DROP ANY TABLE,ALTER ANY TABLE, CREATE VIEW , CREATE SESSION from dev1;

Revoke succeeded.
```

 
  - **Créer des rôles dédiés pour chaque Equipe d'utilisateurs en réspectant les critères suivants** 

      A) Le rôle de l'équipe Dev permet de:

      * Création de procédures stockées.
      * Création de vue.
      * Création de séquence.
      * Création de session.
      * Création,lecture, modification de structure et suppression de tables.
      
      B) Le rôle de l'équipe Test permet de:

      * Se connecter à la base de données.
      * Création de session.
      * Lecture données de toutes les tables.
     
     C) Le rôle de l'équipe DevSecOps permet d'avoir tous les privilèges avec mode administrateur de la base:  

```sql
CREATE ROLE dev_team;

Role created.
grant CREATE PROCEDURE , CREATE SEQUENCE , CREATE ANY TABLE ,SELECT ANY TABLE, DROP ANY TABLE,ALTER ANY TABLE, CREATE VIEW , CREATE SESSION TO dev_team;

Grant succeeded.
```
```sql

```
```sql
SQL> CREATE ROLE test_team;

Role created.
SQL> grant  SELECT ANY TABLE,  CREATE SESSION, CONNECT TO test_team;

Grant succeeded.
```
```sql
SQL> CREATE ROLE devops_team;

Role created.
SQL> grant  DBA TO devops_team;

Grant succeeded.
```


 
   - **Attribuer à chaque utilisateur, le rôle qui lui correspond:** 
  

```sql
SQL> grant dev_team to dev1,dev2;

Grant succeeded.
```
```sql
SQL> grant test_team to tester1,tester2;

Grant succeeded

```
```sql
SQL> grant devops_team to devsecops1,devsecops2;

Grant succeeded.

```
   - **Limiter l'accès pour les testeurs de sorte qu'ils n'accèdent qu'à la table des employés "EMP":** 
  

```sql
revoke select any table from test_team;

Revoke succeeded.
```

 ```sql
SQL> grant  SELECT ON EMP TO test_team;

Grant succeeded.
```
 
 
 
   - **Autoriser tous les utilisateurs sur le système pour interroger les données de la table EMP :** 
  

 ```sql
SQL> grant select on  EMP  to public;

Grant succeeded.
```

**Retirer les privilèges attribuées aux admins, ainsi que les utilisateurs qui ont reçu leurs privilèges sur la table EMP par un membre de l'équipe devsecops:**

 
 
```sql
SQL> connect devsecops1
Enter password:
Connected.

SQL> SQL> REVOKE
  2  DBA
  3  FROM devops_team ;
```


**Créer un profile de ressources dédié à l'équipe des développeurs avec les limitations suivantes:**
  * ***Nombre maximal de sessions permises par utilisateur:*** **illimité**
  * ***Nombre maximal de CPU par session:*** **10000**  
  * ***Nombre maximal de CPU par appel à la base :*** **1000**
  * ***Durée maximal en secondes d'une session:*** ***45*** 
  * ***Nombre maximal de lectures logiques par session utilisateur:*** ***Valeur par defaut***
  * ***Nombre maximal de lectures logiques par appel à la base:*** ***1000***
  * ***Taille maximale de l'SGA privée:*** ***25K***
  * ***Durée de vie en jours du mot de passe:*** ***60***
  * ***Nombre maximal de réutilisations de mot de passe:*** ***10***



```sql 
SQL> CREATE PROFILE PDEV
  2  LIMIT
  3  SESSIONS_PER_USER UNLIMITED
  4  CPU_PER_SESSION 10000
  5  CPU_PER_CALL 1000
  6  CONNECT_TIME 45
  7  LOGICAL_READS_PER_SESSION DEFAULT
  8  LOGICAL_READS_PER_CALL 1000
  9  PRIVATE_SGA 25K
 10  PASSWORD_LIFE_TIME 60
 11  PASSWORD_REUSE_TIME 10;

Profile created.
```




**Créer un profile de ressources dédié à l'équipe de test avec les limitations suivantes:**
  * ***Nombre maximal de sessions permises par utilisateur:*** **5**
  * ***Nombre maximal de CPU par session:*** **illimité**  
  * ***Nombre maximal de CPU par appel à la base :*** **3000**
  * ***Durée maximal en secondes d'une session:*** ***45*** 
  * ***Nombre maximal de lectures logiques par session utilisateur:*** ***Valeur par defaut***
  * ***Nombre maximal de lectures logiques par appel à la base:*** ***1000***
  * ***Taille maximale de l'SGA privée:*** ***25K***
  * ***Durée de vie en jours du mot de passe:*** ***60***
  * ***Nombre maximal de réutilisations de mot de passe:*** ***10***
```sql 
SQL> CREATE PROFILE PDET
  2  LIMIT
  3  SESSIONS_PER_USER 5
  4  CPU_PER_SESSION UNLIMITED
  5  CPU_PER_CALL 3000
  6  CONNECT_TIME 45
  7  LOGICAL_READS_PER_SESSION DEFAULT
  8  LOGICAL_READS_PER_CALL 1000
  9  PRIVATE_SGA 25K
 10  PASSWORD_LIFE_TIME 60
 11  PASSWORD_REUSE_TIME 10;

Profile created.
```

**Créer un profile de ressources dédié à l'équipe devsecops avec les limitations suivantes:**
  * ***Nombre maximal de sessions permises par utilisateur:*** **illimité**
  * ***Nombre maximal de CPU par session:*** **illimité**  
  * ***Nombre maximal de CPU par appel à la base :*** **3000**
  * ***Durée maximal en secondes d'une session:*** ***3600*** 
  * ***Nombre maximal de lectures logiques par session utilisateur:*** ***Valeur par defaut***
  * ***Nombre maximal de lectures logiques par appel à la base:*** ***5000***
  * ***Taille maximale de l'SGA privée:*** ***80K***
  * ***Durée de vie en jours du mot de passe:*** ***60***
  * ***Nombre maximal de réutilisations de mot de passe:*** ***10***

```sql 
CREATE PROFILE PDEVOPS
LIMIT SESSIONS_PER_USER UNLIMITED CPU_PER_SESSION UNLIMITED
CPU_PER_CALL 3000 
CONNECT_TIME 60 
LOGICAL_READS_PER_SESSION DEFAULT
LOGICAL_READS_PER_CALL 5000 
PRIVATE_SGA 80K 
PASSWORD_LIFE_TIME 60 
PASSWORD_REUSE_MAX 10;
```

  - **Attribuer à l'utilisateur "dev1", le profile qui lui correspond:** 
```sql
SQL> ALTER USER dev1 PROFILE PDEV;

User altered.
```


# Objectif de ce document

Décrire le pas à faire pour patcher et installer un paquet log4j 1.2.14 sur CentOS 6.10 libre de la faille de sécurité CVE-2021-4104

Sur un CentOS 6.10, récupérer la dernière version disponible du paquet source log4j : log4j-1.2.14-6.4.el6, l'installer avec toutes ses dépendances et voir les rémédiations nécessaires.

## 1 : Installer une VM avec CentOS 6.10. Exemple de virtualBox comme moteur de virtualisation sur une distribution Linux Mint.
<details><summary>Images de CentOS 6.10 à télécharger depuis le vault centOS</summary>

```
wget -c http://vault.centos.org/6.10/isos/x86_64/CentOS-6.10-x86_64-bin-DVD1.iso
wget -c http://vault.centos.org/6.10/isos/x86_64/CentOS-6.10-x86_64-bin-DVD2.iso
```

</details>

<details><summary>Actions additionnelles pour faire fonctionner la VM</summary>

- Configurer le reseau pour avoir un accès externe indépendant:
    - https://www.serverlab.ca/tutorials/linux/administration-linux/configure-centos-6-network-settings/
- Créer le repo en local à partir de l'image DVD :
    - https://www.itzgeek.com/how-tos/linux/centos-how-tos/create-local-yum-repository-on-centos-7-rhel-7-using-dvd.html
- Ajouter les guest-additions à la VM :
    - https://lecrabeinfo.net/virtualbox-installer-les-additions-invite-guest-additions.html

</details>

## 2 : Télécharger les SRPMs log4j-1.2.17-17.el7_4 et log4j-1.2.14-6.4.el6

Télécharger les SRPMs [log4j-1.2.17-17.el7_4.src.rpm](https://vault.centos.org/7.9.2009/updates/Source/SPackages/log4j-1.2.17-17.el7_4.src.rpm) pour analyse des patches appliqués pour CentOS7 et [log4j-1.2.14-6.4.el6.src.rpm](https://vault.centos.org/6.10/os/Source/SPackages/log4j-1.2.14-6.4.el6.src.rpm) pour application des patches pour CentOS 6

<details>
<summary>log4j-1.2.17-17.el7.src.rpm</summary>

https://vault.centos.org/7.9.2009/updates/Source/SPackages/log4j-1.2.17-17.el7_4.src.rpm

```
wget -c https://vault.centos.org/7.9.2009/updates/Source/SPackages/log4j-1.2.17-17.el7_4.src.rpm
```

</details>

<details>
<summary>log4j-1.2.14-6.4.el6.src.rpm</summary>

https://vault.centos.org/6.10/os/Source/SPackages/log4j-1.2.14-6.4.el6.src.rpm

```
wget -c https://vault.centos.org/6.10/os/Source/SPackages/log4j-1.2.14-6.4.el6.src.rpm
```

</details>

## 3 : Etude de la vulnérabilité dans cette version de log4j

Selon la page officielle d'apache pour les [vulnérabilités](https://logging.apache.org/log4j/2.x/security.html) : 
- CVE-2021-44228 :
>    - Log4j 1.x mitigation  
>Log4j 1.x does not have Lookups so the risk is lower. 
>Applications using Log4j 1.x are only vulnerable to this attack when they use JNDI in their configuration. 
>**A separate CVE (CVE-2021-4104) has been filed for this vulnerability.**
>To mitigate: Audit your logging configuration to ensure it has no JMSAppender configured. 
>Log4j 1.x configurations without JMSAppender are not impacted by this vulnerability.

Dans le repo officiel de apache log4j c'est indiqué que toute contribution pour la branche 1.2.x sera cloturée comme "won't fix" et qu'il faut migrer vers la version 2 :
  - [git apache log4j 1.2.x - README.md](https://github.com/apache/logging-log4j1/blob/main/README.md)

**RedHat** adresse cette faille dans la page son [portail de support](https://access.redhat.com/security/cve/CVE-2021-4104) :
>Note this flaw ONLY affects applications which are specifically configured to use JMSAppender, which is not the default, or when the attacker has write access to the Log4j configuration for adding JMSAppender to the attacker's JNDI LDAP endpoint.

>**Mitigation**
>
>These are the possible mitigations for this flaw for releases version 1.x:
>
>- Comment out or remove JMSAppender in the Log4j configuration if it is used
>- Remove the JMSAppender class from the classpath. For example:
>
>`zip -q -d log4j-*.jar org/apache/log4j/net/JMSAppender.class`
>
>- Restrict access for the OS user on the platform running the application to prevent modifying the Log4j configuration by the attacker.

**La correction sur log4j1 a été faite par RedHat sur le paquet log4j-1.2.17-17.el7_4**

Dans le [changelog](https://centos.pkgs.org/7/centos-updates-x86_64/log4j-1.2.17-17.el7_4.noarch.rpm.html#changelog) de la version patchée de log4j-1.2.17-17.el7_4 on peut lire :

> 2021-12-15 - Mikolaj Izdebski <mizdebsk@redhat.com> - 0:1.2.17-17
> - Fix remote code execution vulnerability
> - Resolves: **CVE-2021-4104**

**Constatation des modifications apportées dans le SRPM de la version 1.2.17-17 pour corriger la CVE-2021-4104**

Dans les sources du rpm [log4j-1.2.17-17_el7_4](http://vault.centos.org/7.9.2009/updates/Source/SPackages/log4j-1.2.17-17.el7_4.src.rpm) on peut voir que la classe JMSAppender a été désactivée par défaut, et pas supprimée du paquetage :

>JNDI, which is used by JMS appender, has significant security issues.
>It is safer for users to disable JMS appender by default,
>especially since the large majority are unlikely to be using it.
>Those who are will need to explicitly enable it, for example:
>
>    `log4j.appender.jms=org.apache.log4j.net.JMSAppender`  
>    `log4j.appender.jms.Enabled=true`
>
>This is a simillar approach to the one implemented in Log4J 2:
>https://issues.apache.org/jira/browse/LOG4J2-3208


## 4 : Analyse des patchs de la version 1.2.17-17.**el7**_4

2 patches ont été appliqués dans la version 1.2.17-17.el7_4 :
- 0001-Add-test-case-for-JNDI-disablement.patch
- 0002-Disable-JNDI-by-default.patch

Le premier est utilisé pour intégrer les tests unitaires pour la modification apportée dans le deuxième, qui désactive par défaut l'appender JMS

Dans le fichier log4j.spec on peut voir l'application de ces 2 patches :

```
...
Patch7:         0001-Add-test-case-for-JNDI-disablement.patch
Patch8:         0002-Disable-JNDI-by-default.patch
...
%patch7 -p1 -b .log4shell
%patch8 -p1 -b .log4shell
...

```

Les patches ne peuvent pas être appliqués directement sur le SRPM de la version 1.2.14 car l'arborescence et le code source ne sont pas exactement pareils. Pour la version 1.2.17 on utilise junit pour faire des tests pendant la compilation du paquet, mais pour la version 1.2.14 on ne l'utilise pas :

- Patch 0001 :
    - 1.2.17 : 
        - `tests/build.xml                               | 14 +++++++-`
        - `tests/input/JNDI0.properties                  |  3 ++`
        - `tests/input/JNDI1.properties                  |  4 +++`
        - `tests/src/java/org/apache/log4j/JNDITestCase.java   | 34 +++++++++++++++++++`
    - 1.2.14 : _**Pas de répertoire tests/ à la base de l'arborescence**_
- Patch 0002 :
    - 1.2.17 :
        - `src/main/java/org/apache/log4j/net/JMSAppender.java    | 15 +++++++++++++++`
    - 1.2.14 :
        - Des travaux à faire sur `src/java/org/apache/log4j/net/JMSAppender.java`. Des différences mineures de code existent

Pour le Patch 0002, il a été adapté au code de la classe JMSAppender.java de la version 1.2.14.

Pour le patch 0001, pour la version log4j-1.2.14-6.4-el6, ce patch ne pourra pas s'appliquer, mais des tests équivalents sont prévus à réaliser manuellement après l'installation du paquet.

## 5 : Création de la version **1.2.14-6.5** : Application du patch pour CentOS6 

Basé sur le wiki CentOS pour reconstruir un paquet ([CentOS HowTo Rebuild SRPM](https://wiki.centos.org/HowTos/RebuildSRPM)):

- [Préparer son environnement de build sur une CentOS6](https://wiki.centos.org/HowTos/SetupRpmBuildEnvironment)
    - Création d'un utilisateur builder s'il n'existe pas dans le système. Toutes les manipulations de reconstruction du paquet seront effectuées avec cet utilisateur.  
        `sudo useradd builder`  
        `sudo passwd builder`  
        `su - builder`  
        `sudo yum install rpm-build redhat-rpm-config rpmdevtools`  
        `rpmdev-setuptree`  
        Cette dernière commande créé l'arborescence **rpmbuild** nécessaire pour la reconstruction du paquet
    - Pour supprimer tous les warnings pendant la reconstruction du paquet il faut créer le user mockbuild :  
        `sudo useradd mockbuild`
- Récupérer les sources de [log4j-1.2.14-6.4-el6](https://vault.centos.org/6.10/os/Source/SPackages/log4j-1.2.14-6.4.el6.src.rpm) :
    - `wget -c https://vault.centos.org/6.10/os/Source/SPackages/log4j-1.2.14-6.4.el6.src.rpm`
- Reconstruir à partir des sources pour vérifier qu'on a toutes les dépendances nécessaires :
    1. `rpmbuild --rebuild <chemin de téléchargement>/log4j-1.2.14-6.4.el6.src.rpm`
    2. Installer toutes les dépendances manquantes. Par exemple, dans le cas des paquets manquants ant, classpathx-jaf, classpathx-mail, mx4j et java-javadoc :  
        `sudo yum install ant classpathx-jaf classpathx-mail mx4j java-javadoc`
    3. Répéter la réconstruction (point 1 de cette liste) jusqu'à ce qu'il n'y a plus d'erreurs  
- Installer les sources pour les modifier et appliquer le patch :  
    `rpm -i <chemin de téléchargement>/log4j-1.2.14-6.4.el6.src.rpm`
- Aller dans le répertoire SPECS et préparer les sources :  
    `cd ~/rpmbuild/SPECS/`  
    `rpmbuild -bp log4j.spec`
- Dans le répertoire BUILD, préparer une image d'origine pour création de patch par comparaison :  
    `cd ~/rpmbuild/BUILD`  
    `cp -a logging-log4j-1.2.14 logging-log4j-1.2.14.orig`
- Modifier le fichier source `JMSAppender.java` selon indiqué dans le patch :  
    <details><summary>Contenu du patch</summary>
    
    ```
    --- logging-log4j-1.2.14.orig/src/java/org/apache/log4j/net/JMSAppender.java	2006-09-14 09:04:25.000000000 +0200
    +++ logging-log4j-1.2.14/src/java/org/apache/log4j/net/JMSAppender.java	2022-01-25 13:21:23.541323902 +0100
    @@ -99,6 +99,7 @@ import javax.naming.NamingException;
    @author Ceki G&uuml;lc&uuml; */
     public class JMSAppender extends AppenderSkeleton {
         
    +  boolean enabled;
       String securityPrincipalName;
       String securityCredentials;
       String initialContextFactoryName;
    @@ -118,6 +119,16 @@ public class JMSAppender extends Appende
       JMSAppender() {
       }
    
    +  public
    +  void setEnabled(boolean enabled) {
    +    this.enabled = enabled;
    +  }
    +
    +  public
    +  boolean getEnabled() {
    +    return enabled;
    +  }
    +
       /**
          The <b>TopicConnectionFactoryBindingName</b> option takes a
          string value. Its value will be used to lookup the appropriate
    @@ -168,6 +179,10 @@ public class JMSAppender extends Appende
        *  Options are activated and become effective only after calling
        *  this method.*/
       public void activateOptions() {
    +    if (!enabled) {
    +      throw new IllegalStateException("JMS appender is disabled by default and must be enabled by setting Enabled=true property    of the appender");
    +    }
    +
         TopicConnectionFactory  topicConnectionFactory;
    
         try {
    ```
    </details>  

    `cd logging-log4j-1.2.14`  
    `vim src/java/org/apache/log4j/net/JMSAppender.java` 

- Revenir dans le répertoire de BUILD et générer le patch :  
    `cd ~/rpmbuild/BUILD`  
    `diff Npru logging-log4j-1.2.14.orig logging-log4j-1.2.14 > 0002-Disable-JNDI-by-default-el6.patch`
- Copier le patch dans les SOURCES et modifier le fichier spec pour y ajouter le patch :  
    `cp 0002-Disable-JNDI-by-default-el6.patch ~/rpmbuild/SOURCES/`  
    `cd ~/rpmbuild/SPECS`  
    `vi log4j.spec` :  

        ...
        Release:        6.5%{?dist}
        ...
        Patch4:         0002-Disable-JNDI-by-default-el6.patch
        ...
        %patch4 -p1 -b .log4shell
        ...
        * Tue Jan 27 2022 Jose V Mondejar <jose-vicente.mondejar@atos.net> - 0:1.2.14-6.5
        - Fix remote code execution vulnerability. Inspired from 1.2.17-17.el7_4 (Mikolaj Izdebski <mizdebsk@redhat.com>)
        - Resolves: CVE-2021-4104
        ...
    
- Construire le nouveau paquet :  
    `rpmbuild -ba log4j.spec`

Les RPMs sont générés dans ~/rpmbuild/RPMS/x86_64 :
```
[builder@centos6 ~]$ ll rpmbuild/RPMS/x86_64/
total 6392
-rw-rw-r--. 1 builder builder  693896 27 janv. 12:53 log4j-1.2.14-6.5.el6.x86_64.rpm
-rw-rw-r--. 1 builder builder  625808 27 janv. 12:53 log4j-debuginfo-1.2.14-6.5.el6.x86_64.rpm
-rw-rw-r--. 1 builder builder  171636 27 janv. 12:53 log4j-javadoc-1.2.14-6.5.el6.x86_64.rpm
-rw-rw-r--. 1 builder builder 1775456 27 janv. 12:53 log4j-manual-1.2.14-6.5.el6.x86_64.rpm
```

Le SRPM est généré dans ~/rpmbuild/SRPMS/ :
```
[builder@centos6 ~]$ ll rpmbuild/SRPMS/
total 2728
-rw-rw-r--. 1 builder builder 2792920 27 janv. 12:18 log4j-1.2.14-6.5.el6.src.rpm
```

## 6 : Installer le paquet log4j-1.2.14-6.5.el6.x86_64.rpm sur CentOS 6.10

- Récupérer le paquet **log4j-1.2.14-6._5_.el6.x86_64.rpm**

- En tant que root, l'installer sur le système à l'aide de yum


    <details>
    <summary>

    `[root@centos610 ~]# yum localinstall <chemin_vers_le_rpm>/log4j-1.2.14-6.5.el6.x86_64.rpm`

    </summary>



    ```
    Modules complémentaires chargés : fastestmirror, security
    Configuration du processus de paquets locaux
    Examen de /mnt/rpmbuild/RPMS/x86_64/log4j-1.2.14-6.5.el6.x86_64.rpm : log4j-1.2.14-6.5.el6.x86_64
    Sélection de /mnt/rpmbuild/RPMS/x86_64/log4j-1.2.14-6.5.el6.x86_64.rpm pour installation 
    Loading mirror speeds from cached hostfile
    Résolution des dépendances
    --> Lancement de la transaction de test
    ---> Package log4j.x86_64 0:1.2.14-6.5.el6 will be installé
    --> Traitement de la dépendance : java-gcj-compat pour le paquet : log4j-1.2.14-6.5.el6.x86_64
    --> Traitement de la dépendance : java-gcj-compat pour le paquet : log4j-1.2.14-6.5.el6.x86_64
    --> Traitement de la dépendance : jaxp_parser_impl pour le paquet : log4j-1.2.14-6.5.el6.x86_64
    --> Traitement de la dépendance : xml-commons-apis pour le paquet : log4j-1.2.14-6.5.el6.x86_64
    --> Lancement de la transaction de test
    ---> Package java-1.5.0-gcj.x86_64 0:1.5.0.0-29.1.el6 will be installé
    --> Traitement de la dépendance : sinjdoc pour le paquet : java-1.5.0-gcj-1.5.0.0-29.1.el6.x86_64
    ---> Package xml-commons-apis.x86_64 0:1.3.04-3.6.el6 will be installé
    --> Lancement de la transaction de test
    ---> Package sinjdoc.x86_64 0:0.5-9.1.el6 will be installé
    --> Traitement de la dépendance : java_cup >= 0.10 pour le paquet : sinjdoc-0.5-9.1.el6.x86_64
    --> Lancement de la transaction de test
    ---> Package java_cup.x86_64 1:0.10k-5.el6 will be installé
    --> Résolution des dépendances terminée

    Dépendances résolues

    ==============================================================================================================================================================================================================================================
     Paquet                                                   Architecture                                   Version                                                   Dépôt                                                                Taille
    ==============================================================================================================================================================================================================================================
    Installation:
     log4j                                                    x86_64                                         1.2.14-6.5.el6                                            /log4j-1.2.14-6.5.el6.x86_64                                         2.0 M
    Installation pour dépendance:
     java-1.5.0-gcj                                           x86_64                                         1.5.0.0-29.1.el6                                          base                                                                 139 k
     java_cup                                                 x86_64                                         1:0.10k-5.el6                                             base                                                                 197 k
     sinjdoc                                                  x86_64                                         0.5-9.1.el6                                               base                                                                 705 k
     xml-commons-apis                                         x86_64                                         1.3.04-3.6.el6                                            base                                                                 439 k

    Résumé de la transaction
    ==============================================================================================================================================================================================================================================
    Installation de     5 paquet(s)

    Taille totale : 3.4 M
    Taille totale des téléchargements : 1.4 M
    Taille d'installation : 6.6 M
    Est-ce correct [o/N] : O
    Téléchargement des paquets :
    (1/4): java-1.5.0-gcj-1.5.0.0-29.1.el6.x86_64.rpm                                                                                                                                                                      | 139 kB     00:00     
    (2/4): java_cup-0.10k-5.el6.x86_64.rpm                                                                                                                                                                                 | 197 kB     00:00     
    (3/4): sinjdoc-0.5-9.1.el6.x86_64.rpm                                                                                                                                                                                  | 705 kB     00:01     
    (4/4): xml-commons-apis-1.3.04-3.6.el6.x86_64.rpm                                                                                                                                                                      | 439 kB     00:00     
    ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
    Total                                                                                                                                                                                                         274 kB/s | 1.4 MB     00:05     
    Lancement de rpm_check_debug
    Lancement de la transaction de test
    Transaction de test réussie
    Lancement de la transaction
      Installation  : java-1.5.0-gcj-1.5.0.0-29.1.el6.x86_64                                                                                                                                                                                  1/5 
      Installation  : 1:java_cup-0.10k-5.el6.x86_64                                                                                                                                                                                           2/5 
      Installation  : sinjdoc-0.5-9.1.el6.x86_64                                                                                                                                                                                              3/5 
      Installation  : xml-commons-apis-1.3.04-3.6.el6.x86_64                                                                                                                                                                                  4/5 
      Installation  : log4j-1.2.14-6.5.el6.x86_64                                                                                                                                                                                             5/5 
      Verifying     : xml-commons-apis-1.3.04-3.6.el6.x86_64                                                                                                                                                                                  1/5 
      Verifying     : 1:java_cup-0.10k-5.el6.x86_64                                                                                                                                                                                           2/5 
      Verifying     : java-1.5.0-gcj-1.5.0.0-29.1.el6.x86_64                                                                                                                                                                                  3/5 
      Verifying     : sinjdoc-0.5-9.1.el6.x86_64                                                                                                                                                                                              4/5 
      Verifying     : log4j-1.2.14-6.5.el6.x86_64                                                                                                                                                                                             5/5 
  
    Installé:
      log4j.x86_64 0:1.2.14-6.5.el6                                                                                                                                                                                                               

    Dépendance(s) installée(s) :
      java-1.5.0-gcj.x86_64 0:1.5.0.0-29.1.el6                         java_cup.x86_64 1:0.10k-5.el6                         sinjdoc.x86_64 0:0.5-9.1.el6                         xml-commons-apis.x86_64 0:1.3.04-3.6.el6                        

    Terminé !
    ```



    </details>

## 7 : Tester le paquet installé sur CentOS 6.10

**- Installation des paquets de dévéloppement et test nécessaires**
```
sudo yum groupinstall "Development Tools"
sudo yum install java-1.5.0-gcj-devel junit ant jakarta-oro geronimo-specs-compat
```

**- Extraction du répertoire de tests de log4j-1.2.17.tar.gz sur un répertoire local**  
- Copier le fichier log4j-1.2.17.tar.gz du répertoire ~/rpmbuild/SOURCES sur un répertoire temporaire et extraire le répertoire tests localement  
- Aller dans le répertoire de tests et créer les sous-répertoires pour la compilation et les tests junit:
```
cd <chemin du répertoire tests>
mkdir target
mkdir target/classes
mkdir output
```

**- Créer les fichiers d'input pour junit:**  

<details>
<summary>JNDI0.properties</summary>

```
log4j.rootLogger=DEBUG, testAppender
log4j.appender.testAppender=org.apache.log4j.net.JMSAppender
log4j.appender.testAppender.TopicConnectionFactoryBindingName=foo
```

</details>

<details>
<summary>JNDI1.properties</summary>

```
log4j.rootLogger=DEBUG, testAppender
log4j.appender.testAppender=org.apache.log4j.net.JMSAppender
log4j.appender.testAppender.TopicConnectionFactoryBindingName=foo
log4j.appender.testAppender.Enabled=true
```

</details>

```
vim input\JNDI1.properties
vim input\JNDI0.properties

```

**- Préparer le classpath pour les tests**  
```
export CLASSPATH=/usr/share/java/jms.jar:/usr/share/java/junit.jar:/usr/share/java/jakarta-oro-2.0.8.jar:target/classes:../../target/classes:resources:/usr/share/java/log4j.jar
```

**- Compiler la classe pour les tests**  
```
[centos@centos610 tests]$ cd src/java
[centos@centos610 java]$ javac -d ../../target/classes org/apache/log4j/JNDITestCase.java 
1. WARNING in /mnt/tests/src/java/org/apache/log4j/JNDITestCase.java (at line 21)
	Logger logger = Logger.getLogger(JNDITestCase.class);
	       ^^^^^^
The value of the local variable logger is not used
----------
2. WARNING in /mnt/tests/src/java/org/apache/log4j/JNDITestCase.java (at line 31)
	Logger logger = Logger.getLogger(JNDITestCase.class);
	       ^^^^^^
The value of the local variable logger is not used
----------
2 problems (2 warnings)

```

Les warnings peuvent être ignorés

**- Effectuer les tests**  
```
[centos@centos610 tests]# java junit.textui.TestRunner org.apache.log4j.JNDITestCase 2> /dev/null
..
Time: 0,083

OK (2 tests)

```

## 8 : Signer les RPMS

**- Génération de la clé GPG**
```
[builder@centos6 ~]$ gpg --gen-key
gpg (GnuPG) 2.0.14; Copyright (C) 2009 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Sélectionnez le type de clé désiré:
   (1) RSA and RSA (default)
   (2) DSA and Elgamal
   (3) DSA (signature seule)
   (4) RSA (signature seule)
Votre choix ? 
les clés RSA peuvent faire entre 1024 et 4096 bits de longueur.
Quelle taille de clé désirez-vous ? (2048) 
La taille demandée est 2048 bits
Spécifiez combien de temps cette clé devrait être valide.
         0 = la clé n'expire pas
      <n>  = la clé expire dans n jours
      <n>w = la clé expire dans n semaines
      <n>m = la clé expire dans n mois
      <n>y = la clé expire dans n années
La clé est valide pour ? (0) 
La clé n'expire pas du tout
Est-ce correct ? (o/N) O

You need a user ID to identify your key; the software constructs the user ID
from the Real Name, Comment and Email Address in this form:
    "Heinrich Heine (Der Dichter) <heinrichh@duesseldorf.de>"

Nom réel: Jose Vicente Mondejar
Adresse e-mail: jose-vicente.mondejar@atos.net
Commentaire: RPM signing key
Vous avez sélectionné ce nom d'utilisateur:
    "Jose Vicente Mondejar (RPM signing key) <jose-vicente.mondejar@atos.net>"

Changer le (N)om, le (C)ommentaire, l'(E)-mail ou (O)K/(Q)uitter ? O
Vous avez besoin d'une phrase de passe pour protéger votre clé
secrète.

can't connect to `/home/builder/.gnupg/S.gpg-agent': Aucun fichier ou dossier de ce type
Un grand nombre d'octets aléatoires doit être généré. Vous devriez faire
autre-chose (taper au clavier, déplacer la souris, utiliser les disques)
pendant la génération de nombres premiers; cela donne au générateur de
nombres aléatoires une meilleure chance d'avoir assez d'entropie.
gpg: /home/builder/.gnupg/trustdb.gpg: base de confiance créée
gpg: clé A60289F2 marquée comme ayant une confiance ultime.
les clés publique et secrète ont été créées et signées.

gpg: vérifier la base de confiance
gpg: 3 marginale(s) nécessaires, 1 complète(s) nécessaires, modèle
de confiance PGP
gpg: profondeur: 0  valide:   1  signé:   0
confiance: 0-. 0g. 0n. 0m. 0f. 1u
pub   2048R/A60289F2 2022-02-24
    Empreinte de la clé = 5323 800D 9065 E13B 13BD  9505 7689 125A A602 89F2
uid                  Jose Vicente Mondejar (RPM signing key) <jose-vicente.mondejar@atos.net>
sub   2048R/11C0E177 2022-02-24

```

**- Vérification de la génération de la clé GPG**
```
[builder@centos6 ~]$ gpg --list-keys
/home/builder/.gnupg/pubring.gpg
--------------------------------
pub   2048R/A60289F2 2022-02-24
uid                  Jose Vicente Mondejar (RPM signing key) <jose-vicente.mondejar@atos.net>
sub   2048R/11C0E177 2022-02-24

```

**- Exportation de la clé publique à communiquer**
```
[builder@centos6 ~]$ gpg --export -a 'Jose Vicente Mondejar' > RPM-GPG-KEY-log4j

```

**- Importation et vérification de la clé publique dans le trousseau RPM**
```
[builder@centos6 ~]$ sudo rpm --import RPM-GPG-KEY-log4j 
[builder@centos6 ~]$ rpm -q gpg-pubkey --qf '%{name}-%{version}-%{release} --> %{summary}\n'
gpg-pubkey-a60289f2-6217e61e --> gpg(Jose Vicente Mondejar (RPM signing key) <jose-vicente.mondejar@atos.net>)

```

**- Modification du fichier .rpmmacros**
```
[builder@centos6 ~]$ vim .rpmmacros 
[builder@centos6 ~]$ cat .rpmmacros 
%_topdir      %(echo $HOME)/rpmbuild
%_smp_mflags  -j3
%__arch_install_post   /usr/lib/rpm/check-rpaths   /usr/lib/rpm/check-buildroot
%_signature gpg
%_gpg_path /home/builder/.gnupg
%_gpg_name Jose Vicente Mondejar
%_gpgbin /usr/bin/gpg2
%__gpg_sign_cmd %{__gpg} gpg --force-v3-sigs --batch --verbose --no-armor --passphrase-fd 3 --no-secmem-warning -u "%{_gpg_name}" -sbo %{__signature_filename} --digest-algo sha256 %{__plaintext_filename}'

```

**- Signature des RPMs**
```
[builder@centos6 ~]$ cd rpmbuild/RPMS/x86_64/
[builder@centos6 x86_64]$ rpm --addsign log4j-*1.2.14-6.5.el6.x86_64.rpm
Entrez la phrase de passe: 
Phrase de passe bonne.
log4j-1.2.14-6.5.el6.x86_64.rpm:
gpg: writing to `/var/tmp/rpm-tmp.KLdkxh.sig'
gpg: RSA/SHA256 signature from: "A60289F2 Jose Vicente Mondejar (RPM signing key) <jose-vicente.mondejar@atos.net>"
gpg: writing to `/var/tmp/rpm-tmp.UGCzvl.sig'
gpg: RSA/SHA256 signature from: "A60289F2 Jose Vicente Mondejar (RPM signing key) <jose-vicente.mondejar@atos.net>"
log4j-debuginfo-1.2.14-6.5.el6.x86_64.rpm:
gpg: writing to `/var/tmp/rpm-tmp.M5oVqx.sig'
gpg: RSA/SHA256 signature from: "A60289F2 Jose Vicente Mondejar (RPM signing key) <jose-vicente.mondejar@atos.net>"
gpg: writing to `/var/tmp/rpm-tmp.QQhErF.sig'
gpg: RSA/SHA256 signature from: "A60289F2 Jose Vicente Mondejar (RPM signing key) <jose-vicente.mondejar@atos.net>"
log4j-javadoc-1.2.14-6.5.el6.x86_64.rpm:
gpg: writing to `/var/tmp/rpm-tmp.UEJ8sZ.sig'
gpg: RSA/SHA256 signature from: "A60289F2 Jose Vicente Mondejar (RPM signing key) <jose-vicente.mondejar@atos.net>"
gpg: writing to `/var/tmp/rpm-tmp.O9ietb.sig'
gpg: RSA/SHA256 signature from: "A60289F2 Jose Vicente Mondejar (RPM signing key) <jose-vicente.mondejar@atos.net>"
log4j-manual-1.2.14-6.5.el6.x86_64.rpm:
gpg: writing to `/var/tmp/rpm-tmp.g03JsD.sig'
gpg: RSA/SHA256 signature from: "A60289F2 Jose Vicente Mondejar (RPM signing key) <jose-vicente.mondejar@atos.net>"
gpg: writing to `/var/tmp/rpm-tmp.iNT7tT.sig'
gpg: RSA/SHA256 signature from: "A60289F2 Jose Vicente Mondejar (RPM signing key) <jose-vicente.mondejar@atos.net>"

```

**- Vérification de la signature des RPMs**
```
[builder@centos6 x86_64]$ rpm --checksig log4j-*1.2.14-6.5.el6.x86_64.rpm 
log4j-1.2.14-6.5.el6.x86_64.rpm: rsa sha1 (md5) pgp md5 OK
log4j-debuginfo-1.2.14-6.5.el6.x86_64.rpm: rsa sha1 (md5) pgp md5 OK
log4j-javadoc-1.2.14-6.5.el6.x86_64.rpm: rsa sha1 (md5) pgp md5 OK
log4j-manual-1.2.14-6.5.el6.x86_64.rpm: rsa sha1 (md5) pgp md5 OK

[builder@centos6 x86_64]$ rpm -q --qf '%{SIGPGP:pgpsig} %{SIGPGP:pgpsig}\n' -p log4j-*1.2.14-6.5.el6.x86_64.rpm 
RSA/8, jeu. 24 févr. 2022 21:44:19 CET, Key ID 7689125aa60289f2 RSA/8, jeu. 24 févr. 2022 21:44:19 CET, Key ID 7689125aa60289f2
RSA/8, jeu. 24 févr. 2022 21:44:21 CET, Key ID 7689125aa60289f2 RSA/8, jeu. 24 févr. 2022 21:44:21 CET, Key ID 7689125aa60289f2
RSA/8, jeu. 24 févr. 2022 21:44:23 CET, Key ID 7689125aa60289f2 RSA/8, jeu. 24 févr. 2022 21:44:23 CET, Key ID 7689125aa60289f2
RSA/8, jeu. 24 févr. 2022 21:44:24 CET, Key ID 7689125aa60289f2 RSA/8, jeu. 24 févr. 2022 21:44:24 CET, Key ID 7689125aa60289f2

```

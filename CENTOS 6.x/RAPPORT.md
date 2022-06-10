# Rapport d'installation de log4j-1.2.14-6.5.el6 sur CentOS 10

`[root@centos610 ~]# yum localinstall <chemin_vers_le_rpm>/log4j-1.2.14-6.5.el6.x86_64.rpm`

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




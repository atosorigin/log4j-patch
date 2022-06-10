# Installation de logstash 6.1.1

Cet exemple montre comment installer logstash sous /opt/ mais un autre répertoire peut-être choisi

```
cd /opt/
tar -xzvf {chemin}/logstash-6.1.1-SNAPSHOT.tar.gz
```

# Lancement de logstash 6.1.1

Créer le fichier de configuration de pipeline (ceci est un exemple) :

```
mkdir -p /etc/logstash/conf.d/
printf 'input { stdin { } }\noutput { stdout { } }' > /etc/logstash/conf.d/logstash-simple.conf
```


Lancer logstash en spécifiant le fichier de configutation :

`/opt/logstash-6.1.1-SNAPSHOT/bin/logstash -f /etc/logstash/conf.d/logstash-simple.conf`

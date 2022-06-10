# Installation de logstash 5.4.0

Cet exemple montre comment installer logstash sous /opt/ mais un autre répertoire peut-être choisi

```
cd /opt/
tar -xzvf {chemin}/logstash-5.4.0-SNAPSHOT.tar.gz
```

# Lancement de logstash 5.4.0

Créer le fichier de configuration de pipeline (ceci est un exemple) :

```
mkdir -p /etc/logstash/conf.d/
printf 'input { stdin { } }\noutput { stdout { } }' > /etc/logstash/conf.d/logstash-simple.conf
```


Lancer logstash en spécifiant le fichier de configutation :

`/opt/logstash-5.4.0-SNAPSHOT/bin/logstash -f /etc/logstash/conf.d/logstash-simple.conf`


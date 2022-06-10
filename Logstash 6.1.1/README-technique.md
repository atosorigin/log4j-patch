# Installation du JDK 8 Oracle

`yum install -y jdk-8u202-linux-x64.rpm`

# Installation de Ruby 2.3

```
yum -y install centos-release-scl-rh
yum -y install rh-ruby23
scl enable rh-ruby23 bash
```


# Installation de Rake

`gem install rake`

# Récupération de la version 6.1.1 à partir de Github

```
git clone --depth 1 --branch v6.1.1 https://github.com/elastic/logstash.git
cd logstash/
```

# Remplacer la version 2.6.2 de log4j par la version 2.17.1

Remplacer toutes les occurrences de "2.6.2" par "2.17.1" dans le fichier logstash-core/build.gradle :
`sed -i 's/2.6.2/2.17.1/g' logstash-core/build.gradle`

# Construire l'archive à partir des sources

`rake artifact:tar`

L'archive construit se trouve dans le répertoire build.

# Tests fonctionnels

Créer et lancer un pipeline basique :

`bin/logstash -e 'input { stdin { } } output { stdout {} }'`

Attendre que logstash ait démarré :

```
Sending Logstash's logs to /opt/logstash-6.1.1-SNAPSHOT/logs which is now configured via log4j2.properties
[2022-01-24T11:57:52,331][INFO ][logstash.modules.scaffold] Initializing module {:module_name=>"fb_apache", :directory=>"/opt/logstash-6.1.1-SNAPSHOT/modules/fb_apache/configuration"}
[2022-01-24T11:57:52,374][INFO ][logstash.modules.scaffold] Initializing module {:module_name=>"netflow", :directory=>"/opt/logstash-6.1.1-SNAPSHOT/modules/netflow/configuration"}
[2022-01-24T11:57:53,879][WARN ][logstash.config.source.multilocal] Ignoring the 'pipelines.yml' file because modules or command line options are specified
[2022-01-24T11:57:55,887][INFO ][logstash.runner          ] Starting Logstash {"logstash.version"=>"6.1.1"}
[2022-01-24T11:57:57,103][INFO ][logstash.agent           ] Successfully started Logstash API endpoint {:port=>9600}
[2022-01-24T11:58:01,465][INFO ][logstash.pipeline        ] Starting pipeline {:pipeline_id=>"main", "pipeline.workers"=>1, "pipeline.batch.size"=>125, "pipeline.batch.delay"=>5, "pipeline.max_inflight"=>125, :thread=>"#<Thread:0x23a68d11 run>"}
[2022-01-24T11:58:01,650][INFO ][logstash.pipeline        ] Pipeline started {"pipeline.id"=>"main"}
The stdin plugin is now waiting for input:
[2022-01-24T11:58:01,850][INFO ][logstash.agent           ] Pipelines running {:count=>1, :pipelines=>["main"]}
```

Taper hello world :

```
hello world
2022-01-24T10:58:13.756Z pc-31.home hello world
```

Quitter Logstash avec CTRL-D :

`[2022-01-24T12:03:58,121][INFO ][logstash.pipeline        ] Pipeline terminated {"pipeline.id"=>"main"}`

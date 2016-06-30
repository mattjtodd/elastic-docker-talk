# COP Elastic & Docker

## Slides 

[Elasticsearch & Docker Swarm](Elasticsearch & Docker Swarm.pdf)

## Docker Compose Elastic Cluster

### Overview

As part of the talk, a three node docker elasticsearch cluster with marvel monitoring and Kibana was demonstrated.  This  was stood up using a [docker compose file] (docker-compose.yml).  This 'infrastructure as code' approache enables reliable reproduction of complete systems independent of host configuration.  Docker makes it easy to bring subsystems in / out of service as part of this group to for example simulate network partitions or perform rolling updates.

The steps described below will create a local application stack in which all of the conatiners will be running on a single host.  In a muti-host setup, the manager will provision the stack on the available swarm salve hosts automatically according to a set of configurable rules.

More information about Docker Swarm can be found [here.] (https://docs.docker.com/swarm/overview/)

### Stand uP!

Follow these steps to get a local stack build and stood up:

* [Install docker for you machine] (https://docs.docker.com/engine/installation/) (forget a about virtual box, it's history) :smile:
* Open a terminal, clone this repo and cd into it
* Enter `docker-compse up`
* Open your browser and go to `http://localhost:5601/app/marvel`
* You can see the cluster which will have the name `elasticsearch`  click this to see the node and index stats.

That's it!

Here's what's happening (can be seen in the logs):

```
Creating network "elasticdockertalk_elastic" with the default driver
```

> Docker is creating a new user-defined network to attach running containers.  This network includes a DNS server to allow services to reference eachother using the name defained in the docker-compose file's service section.  For example `node1`.  More info [here] (https://docs.docker.com/engine/userguide/networking/dockernetworks/)

```
Creating volume "elasticdockertalk_node1" with default driver
Creating volume "elasticdockertalk_node3" with default driver
Creating volume "elasticdockertalk_node2" with default driver
```

> This creates the docker volume to hold data for each of the elastic nodes.  More info [here] (https://docs.docker.com/v1.10/engine/userguide/containers/dockervolumes/).

```
Building node1
Step 1 : FROM elasticsearch:2.3.3
 ---> 6fd640b8fafc
Step 2 : WORKDIR /usr/share/elasticsearch
 ---> Running in 028b1702844f
 ---> 7a5babd23b3e
Removing intermediate container 028b1702844f
Step 3 : RUN gosu elasticsearch bin/plugin install analysis-icu 	&& bin/plugin install https://github.com/elastic/elasticsearch-migration/releases/download/v2.0-alpha2/elasticsearch-migration-2.0-alpha2.zip 	&& gosu elasticsearch bin/plugin install license 	&& gosu elasticsearch bin/plugin install marvel-agent
 ---> Running in eefc5fadd20d
-> Installing analysis-icu...
Trying https://download.elastic.co/elasticsearch/release/org/elasticsearch/plugin/analysis-icu/2.3.3/analysis-icu-2.3.3.zip ...
Downloading .............................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................DONE
Verifying https://download.elastic.co/elasticsearch/release/org/elasticsearch/plugin/analysis-icu/2.3.3/analysis-icu-2.3.3.zip checksums if available ...
Downloading .DONE
Installed analysis-icu into /usr/share/elasticsearch/plugins/analysis-icu
-> Installing from https://github.com/elastic/elasticsearch-migration/releases/download/v2.0-alpha2/elasticsearch-migration-2.0-alpha2.zip...
Trying https://github.com/elastic/elasticsearch-migration/releases/download/v2.0-alpha2/elasticsearch-migration-2.0-alpha2.zip ...
Downloading ...............................DONE
Verifying https://github.com/elastic/elasticsearch-migration/releases/download/v2.0-alpha2/elasticsearch-migration-2.0-alpha2.zip checksums if available ...
Downloading .DONE
Installed elasticsearch-migration into /usr/share/elasticsearch/plugins/elasticsearch-migration
-> Installing license...
Trying https://download.elastic.co/elasticsearch/release/org/elasticsearch/plugin/license/2.3.3/license-2.3.3.zip ...
Downloading .......DONE
Verifying https://download.elastic.co/elasticsearch/release/org/elasticsearch/plugin/license/2.3.3/license-2.3.3.zip checksums if available ...
Downloading .DONE
Installed license into /usr/share/elasticsearch/plugins/license
-> Installing marvel-agent...
Trying https://download.elastic.co/elasticsearch/release/org/elasticsearch/plugin/marvel-agent/2.3.3/marvel-agent-2.3.3.zip ...
Downloading ..........DONE
Verifying https://download.elastic.co/elasticsearch/release/org/elasticsearch/plugin/marvel-agent/2.3.3/marvel-agent-2.3.3.zip checksums if available ...
Downloading .DONE
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@     WARNING: plugin requires additional permissions     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
* java.lang.RuntimePermission setFactory
* javax.net.ssl.SSLPermission setHostnameVerifier
See http://docs.oracle.com/javase/8/docs/technotes/guides/security/permissions.html
for descriptions of what these permissions allow and the associated risks.
Installed marvel-agent into /usr/share/elasticsearch/plugins/marvel-agent
 ---> a56f63029ff6
Removing intermediate container eefc5fadd20d
Step 4 : COPY elasticsearch.yml config
 ---> 8b1dc4438037
Removing intermediate container 5ab8c811a22b
Successfully built 8b1dc4438037
WARNING: Image for service node1 was built because it did not already exist. To rebuild this image you must use `docker-compose build` or `docker-compose up --build`.
Building kibana
Step 1 : FROM kibana:latest
 ---> 298836bc4170
Step 2 : RUN gosu kibana kibana plugin -i elasticsearch/marvel/2.3.3 	&& gosu kibana kibana plugin -i elastic/sense
 ---> Running in b791a97e86e2
Installing marvel
Attempting to transfer from https://download.elastic.co/elasticsearch/marvel/marvel-2.3.3.tar.gz
Transferring 1597693 bytes....................
Transfer complete
Extracting plugin archive
Extraction complete
Optimizing and caching browser bundles...
Plugin installation complete
Installing sense
Attempting to transfer from https://download.elastic.co/elastic/sense/sense-latest.tar.gz
Transferring 1352402 bytes....................
Transfer complete
Extracting plugin archive
Extraction complete
Optimizing and caching browser bundles...
Plugin installation complete
 ---> 588e31830759
Removing intermediate container b791a97e86e2
Successfully built 588e31830759
WARNING: Image for service kibana was built because it did not already exist. To rebuild this image you must use `docker-compose build` or `docker-compose up --build`.
```
> This is building the images will be ran from the container.  The base images for Elasticsearch and Kibana derived from the Docker hub are used as a base and extened to install the plugins required for monitoring and tooling.  These are stored locally once built.

```
Creating elasticdockertalk_node2_1
Creating elasticdockertalk_node3_1
Creating elasticdockertalk_node1_1
Creating elasticdockertalk_kibana_1
Creating elasticdockertalk_balance_1
```

> Creates a container from each of the images using the supplied configuration in the compose file

```
Attaching to elasticdockertalk_node2_1, elasticdockertalk_node3_1, elasticdockertalk_node1_1, elasticdockertalk_kibana_1, elasticdockertalk_balance_1
```

> Attatches each of the container to the network created earlier.

The next log is from the conatiners themselves starting up.  We can see the ES nodes starting and connecting to each other. 

```
node2_1    | [2016-06-30 09:35:24,267][WARN ][bootstrap                ] unable to install syscall filter: seccomp unavailable: your kernel is buggy and you should upgrade
node2_1    | [2016-06-30 09:35:24,623][INFO ][node                     ] [node2] version[2.3.3], pid[1], build[218bdf1/2016-05-17T15:40:04Z]
node2_1    | [2016-06-30 09:35:24,624][INFO ][node                     ] [node2] initializing ...
node3_1    | [2016-06-30 09:35:24,232][WARN ][bootstrap                ] unable to install syscall filter: seccomp unavailable: your kernel is buggy and you should upgrade
node3_1    | [2016-06-30 09:35:24,658][INFO ][node                     ] [node3] version[2.3.3], pid[1], build[218bdf1/2016-05-17T15:40:04Z]
node3_1    | [2016-06-30 09:35:24,658][INFO ][node                     ] [node3] initializing ...
node1_1    | [2016-06-30 09:35:26,689][WARN ][bootstrap                ] unable to install syscall filter: seccomp unavailable: your kernel is buggy and you should upgrade
node3_1    | [2016-06-30 09:35:26,829][INFO ][plugins                  ] [node3] modules [reindex, lang-expression, lang-groovy], plugins [license, analysis-icu, marvel-agent, elasticsearch-migration], sites [elasticsearch-migration]
node2_1    | [2016-06-30 09:35:26,953][INFO ][plugins                  ] [node2] modules [reindex, lang-expression, lang-groovy], plugins [license, analysis-icu, marvel-agent, elasticsearch-migration], sites [elasticsearch-migration]
node3_1    | [2016-06-30 09:35:26,970][INFO ][env                      ] [node3] using [1] data paths, mounts [[/usr/share/elasticsearch/data (/dev/vda2)]], net usable_space [54.6gb], net total_space [59gb], spins? [possibly], types [ext4]
node3_1    | [2016-06-30 09:35:26,973][INFO ][env                      ] [node3] heap size [1007.3mb], compressed ordinary object pointers [true]
node2_1    | [2016-06-30 09:35:27,098][INFO ][env                      ] [node2] using [1] data paths, mounts [[/usr/share/elasticsearch/data (/dev/vda2)]], net usable_space [54.6gb], net total_space [59gb], spins? [possibly], types [ext4]
node2_1    | [2016-06-30 09:35:27,098][INFO ][env                      ] [node2] heap size [1007.3mb], compressed ordinary object pointers [true]
node1_1    | [2016-06-30 09:35:27,461][INFO ][node                     ] [node1] version[2.3.3], pid[1], build[218bdf1/2016-05-17T15:40:04Z]
node1_1    | [2016-06-30 09:35:27,461][INFO ][node                     ] [node1] initializing ...
balance_1  | [2016-06-30 09:35:28,018][WARN ][bootstrap                ] unable to install syscall filter: seccomp unavailable: your kernel is buggy and you should upgrade
balance_1  | [2016-06-30 09:35:28,716][INFO ][node                     ] [balance] version[2.3.3], pid[1], build[218bdf1/2016-05-17T15:40:04Z]
balance_1  | [2016-06-30 09:35:28,716][INFO ][node                     ] [balance] initializing ...
node1_1    | [2016-06-30 09:35:29,952][INFO ][plugins                  ] [node1] modules [reindex, lang-expression, lang-groovy], plugins [license, analysis-icu, marvel-agent, elasticsearch-migration], sites [elasticsearch-migration]
node1_1    | [2016-06-30 09:35:30,066][INFO ][env                      ] [node1] using [1] data paths, mounts [[/usr/share/elasticsearch/data (/dev/vda2)]], net usable_space [54.6gb], net total_space [59gb], spins? [possibly], types [ext4]
node1_1    | [2016-06-30 09:35:30,067][INFO ][env                      ] [node1] heap size [1007.3mb], compressed ordinary object pointers [true]
kibana_1   | {"type":"log","@timestamp":"2016-06-30T09:35:30+00:00","tags":["status","plugin:kibana","info"],"pid":9,"name":"plugin:kibana","state":"green","message":"Status changed from uninitialized to green - Ready","prevState":"uninitialized","prevMsg":"uninitialized"}
kibana_1   | {"type":"log","@timestamp":"2016-06-30T09:35:30+00:00","tags":["status","plugin:elasticsearch","info"],"pid":9,"name":"plugin:elasticsearch","state":"yellow","message":"Status changed from uninitialized to yellow - Waiting for Elasticsearch","prevState":"uninitialized","prevMsg":"uninitialized"}
kibana_1   | {"type":"log","@timestamp":"2016-06-30T09:35:30+00:00","tags":["status","plugin:marvel","info"],"pid":9,"name":"plugin:marvel","state":"yellow","message":"Status changed from uninitialized to yellow - Waiting for Elasticsearch","prevState":"uninitialized","prevMsg":"uninitialized"}
kibana_1   | {"type":"log","@timestamp":"2016-06-30T09:35:31+00:00","tags":["error","elasticsearch"],"pid":9,"message":"Request error, retrying -- connect ECONNREFUSED 172.21.0.6:9200"}
kibana_1   | {"type":"log","@timestamp":"2016-06-30T09:35:31+00:00","tags":["status","plugin:sense","info"],"pid":9,"name":"plugin:sense","state":"green","message":"Status changed from uninitialized to green - Ready","prevState":"uninitialized","prevMsg":"uninitialized"}
kibana_1   | {"type":"log","@timestamp":"2016-06-30T09:35:31+00:00","tags":["status","plugin:kbn_vislib_vis_types","info"],"pid":9,"name":"plugin:kbn_vislib_vis_types","state":"green","message":"Status changed from uninitialized to green - Ready","prevState":"uninitialized","prevMsg":"uninitialized"}
kibana_1   | {"type":"log","@timestamp":"2016-06-30T09:35:31+00:00","tags":["warning","elasticsearch"],"pid":9,"message":"Unable to revive connection: http://balance:9200/"}
kibana_1   | {"type":"log","@timestamp":"2016-06-30T09:35:31+00:00","tags":["warning","elasticsearch"],"pid":9,"message":"No living connections"}
kibana_1   | {"type":"log","@timestamp":"2016-06-30T09:35:31+00:00","tags":["status","plugin:elasticsearch","error"],"pid":9,"name":"plugin:elasticsearch","state":"red","message":"Status changed from yellow to red - Unable to connect to Elasticsearch at http://balance:9200.","prevState":"yellow","prevMsg":"Waiting for Elasticsearch"}
kibana_1   | {"type":"log","@timestamp":"2016-06-30T09:35:31+00:00","tags":["status","plugin:markdown_vis","info"],"pid":9,"name":"plugin:markdown_vis","state":"green","message":"Status changed from uninitialized to green - Ready","prevState":"uninitialized","prevMsg":"uninitialized"}
kibana_1   | {"type":"log","@timestamp":"2016-06-30T09:35:31+00:00","tags":["status","plugin:metric_vis","info"],"pid":9,"name":"plugin:metric_vis","state":"green","message":"Status changed from uninitialized to green - Ready","prevState":"uninitialized","prevMsg":"uninitialized"}
balance_1  | [2016-06-30 09:35:31,417][INFO ][plugins                  ] [balance] modules [reindex, lang-expression, lang-groovy], plugins [license, analysis-icu, marvel-agent, elasticsearch-migration], sites [elasticsearch-migration]
kibana_1   | {"type":"log","@timestamp":"2016-06-30T09:35:35+00:00","tags":["status","plugin:spyModes","info"],"pid":9,"name":"plugin:spyModes","state":"green","message":"Status changed from uninitialized to green - Ready","prevState":"uninitialized","prevMsg":"uninitialized"}
kibana_1   | {"type":"log","@timestamp":"2016-06-30T09:35:35+00:00","tags":["status","plugin:statusPage","info"],"pid":9,"name":"plugin:statusPage","state":"green","message":"Status changed from uninitialized to green - Ready","prevState":"uninitialized","prevMsg":"uninitialized"}
kibana_1   | {"type":"log","@timestamp":"2016-06-30T09:35:35+00:00","tags":["status","plugin:table_vis","info"],"pid":9,"name":"plugin:table_vis","state":"green","message":"Status changed from uninitialized to green - Ready","prevState":"uninitialized","prevMsg":"uninitialized"}
kibana_1   | {"type":"log","@timestamp":"2016-06-30T09:35:35+00:00","tags":["warning","elasticsearch"],"pid":9,"message":"Unable to revive connection: http://balance:9200/"}
kibana_1   | {"type":"log","@timestamp":"2016-06-30T09:35:35+00:00","tags":["warning","elasticsearch"],"pid":9,"message":"No living connections"}
kibana_1   | {"type":"log","@timestamp":"2016-06-30T09:35:35+00:00","tags":["listening","info"],"pid":9,"message":"Server running at http://0.0.0.0:5601"}
node2_1    | [2016-06-30 09:35:36,476][INFO ][node                     ] [node2] initialized
node2_1    | [2016-06-30 09:35:36,477][INFO ][node                     ] [node2] starting ...
node2_1    | [2016-06-30 09:35:36,481][DEBUG][marvel.agent.collector.node] [node2] starting collector [node-stats-collector]
node2_1    | [2016-06-30 09:35:36,481][DEBUG][marvel.agent.collector.cluster] [node2] starting collector [cluster-stats-collector]
node2_1    | [2016-06-30 09:35:36,482][DEBUG][marvel.agent.collector.shards] [node2] starting collector [shards-collector]
node2_1    | [2016-06-30 09:35:36,482][DEBUG][marvel.agent.collector.indices] [node2] starting collector [index-recovery-collector]
node2_1    | [2016-06-30 09:35:36,483][DEBUG][marvel.agent.collector.indices] [node2] starting collector [indices-stats-collector]
node2_1    | [2016-06-30 09:35:36,484][DEBUG][marvel.agent.collector.indices] [node2] starting collector [index-stats-collector]
node2_1    | [2016-06-30 09:35:36,484][DEBUG][marvel.agent.collector.cluster] [node2] starting collector [cluster-state-collector]
node2_1    | [2016-06-30 09:35:36,492][DEBUG][marvel.cleaner           ] [node2] starting cleaning service
node2_1    | [2016-06-30 09:35:36,498][DEBUG][marvel.cleaner           ] [node2] cleaning service started
node3_1    | [2016-06-30 09:35:36,789][INFO ][node                     ] [node3] initialized
node3_1    | [2016-06-30 09:35:36,803][INFO ][node                     ] [node3] starting ...
node3_1    | [2016-06-30 09:35:36,805][DEBUG][marvel.agent.collector.node] [node3] starting collector [node-stats-collector]
node3_1    | [2016-06-30 09:35:36,805][DEBUG][marvel.agent.collector.cluster] [node3] starting collector [cluster-stats-collector]
node3_1    | [2016-06-30 09:35:36,805][DEBUG][marvel.agent.collector.shards] [node3] starting collector [shards-collector]
node3_1    | [2016-06-30 09:35:36,805][DEBUG][marvel.agent.collector.indices] [node3] starting collector [index-recovery-collector]
node3_1    | [2016-06-30 09:35:36,806][DEBUG][marvel.agent.collector.indices] [node3] starting collector [indices-stats-collector]
node3_1    | [2016-06-30 09:35:36,806][DEBUG][marvel.agent.collector.indices] [node3] starting collector [index-stats-collector]
node3_1    | [2016-06-30 09:35:36,806][DEBUG][marvel.agent.collector.cluster] [node3] starting collector [cluster-state-collector]
node3_1    | [2016-06-30 09:35:36,827][DEBUG][marvel.cleaner           ] [node3] starting cleaning service
node3_1    | [2016-06-30 09:35:36,855][DEBUG][marvel.cleaner           ] [node3] cleaning service started
node2_1    | [2016-06-30 09:35:36,926][INFO ][transport                ] [node2] publish_address {172.21.0.2:9300}, bound_addresses {[::]:9300}
node2_1    | [2016-06-30 09:35:36,944][INFO ][discovery                ] [node2] elasticsearch/NlRMBdL1TYSF0049AwX5Vg
node3_1    | [2016-06-30 09:35:37,157][INFO ][transport                ] [node3] publish_address {172.21.0.3:9300}, bound_addresses {[::]:9300}
node3_1    | [2016-06-30 09:35:37,167][INFO ][discovery                ] [node3] elasticsearch/aZC7TyMjTDSK1Iq-LnLYkA
kibana_1   | {"type":"log","@timestamp":"2016-06-30T09:35:37+00:00","tags":["error","elasticsearch"],"pid":9,"message":"Request error, retrying -- connect ECONNREFUSED 172.21.0.6:9200"}
kibana_1   | {"type":"log","@timestamp":"2016-06-30T09:35:37+00:00","tags":["error","elasticsearch"],"pid":9,"message":"Request error, retrying -- connect ECONNREFUSED 172.21.0.6:9200"}
kibana_1   | {"type":"log","@timestamp":"2016-06-30T09:35:37+00:00","tags":["warning","elasticsearch"],"pid":9,"message":"Unable to revive connection: http://balance:9200/"}
kibana_1   | {"type":"log","@timestamp":"2016-06-30T09:35:37+00:00","tags":["warning","elasticsearch"],"pid":9,"message":"No living connections"}
kibana_1   | {"type":"response","@timestamp":"2016-06-30T09:35:37+00:00","tags":[],"pid":9,"method":"post","statusCode":400,"req":{"url":"/api/marvel/v1/clusters/JK3dEr9DR9ePDoVn4LTbBw/indices/.marvel-es-1-2016.06.30","method":"post","headers":{"host":"localhost:5601","connection":"keep-alive","content-length":"218","accept":"application/json, text/plain, */*","origin":"http://localhost:5601","kbn-version":"4.5.1","user-agent":"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_5) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/51.0.2704.103 Safari/537.36","content-type":"application/json;charset=UTF-8","referer":"http://localhost:5601/app/marvel","accept-encoding":"gzip, deflate","accept-language":"en-US,en;q=0.8","cookie":"__hstc=181257784.a4c6e4d089b7432f274a89ef2e621d7d.1463470080164.1463470080164.1463470080164.1; hsfirstvisit=http%3A%2F%2Flocalhost%3A9001%2Flogin.htm%3FredirectTo%3D%2Fapp.htm||1463470080161; hubspotutk=a4c6e4d089b7432f274a89ef2e621d7d"},"remoteAddress":"172.21.0.1","userAgent":"172.21.0.1","referer":"http://localhost:5601/app/marvel"},"res":{"statusCode":400,"responseTime":125,"contentLength":9},"message":"POST /api/marvel/v1/clusters/JK3dEr9DR9ePDoVn4LTbBw/indices/.marvel-es-1-2016.06.30 400 125ms - 9.0B"}
kibana_1   | {"type":"log","@timestamp":"2016-06-30T09:35:37+00:00","tags":["warning","elasticsearch"],"pid":9,"message":"Unable to revive connection: http://balance:9200/"}
kibana_1   | {"type":"log","@timestamp":"2016-06-30T09:35:37+00:00","tags":["warning","elasticsearch"],"pid":9,"message":"No living connections"}
kibana_1   | {"type":"response","@timestamp":"2016-06-30T09:35:37+00:00","tags":[],"pid":9,"method":"post","statusCode":400,"req":{"url":"/api/marvel/v1/clusters","method":"post","headers":{"host":"localhost:5601","connection":"keep-alive","content-length":"81","accept":"application/json, text/plain, */*","origin":"http://localhost:5601","kbn-version":"4.5.1","user-agent":"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_5) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/51.0.2704.103 Safari/537.36","content-type":"application/json;charset=UTF-8","referer":"http://localhost:5601/app/marvel","accept-encoding":"gzip, deflate","accept-language":"en-US,en;q=0.8","cookie":"__hstc=181257784.a4c6e4d089b7432f274a89ef2e621d7d.1463470080164.1463470080164.1463470080164.1; hsfirstvisit=http%3A%2F%2Flocalhost%3A9001%2Flogin.htm%3FredirectTo%3D%2Fapp.htm||1463470080161; hubspotutk=a4c6e4d089b7432f274a89ef2e621d7d"},"remoteAddress":"172.21.0.1","userAgent":"172.21.0.1","referer":"http://localhost:5601/app/marvel"},"res":{"statusCode":400,"responseTime":118,"contentLength":9},"message":"POST /api/marvel/v1/clusters 400 118ms - 9.0B"}
kibana_1   | {"type":"log","@timestamp":"2016-06-30T09:35:37+00:00","tags":["warning","elasticsearch"],"pid":9,"message":"Unable to revive connection: http://balance:9200/"}
kibana_1   | {"type":"log","@timestamp":"2016-06-30T09:35:37+00:00","tags":["warning","elasticsearch"],"pid":9,"message":"No living connections"}
node1_1    | [2016-06-30 09:35:39,048][INFO ][node                     ] [node1] initialized
node1_1    | [2016-06-30 09:35:39,048][INFO ][node                     ] [node1] starting ...
node1_1    | [2016-06-30 09:35:39,050][DEBUG][marvel.agent.collector.node] [node1] starting collector [node-stats-collector]
node1_1    | [2016-06-30 09:35:39,050][DEBUG][marvel.agent.collector.cluster] [node1] starting collector [cluster-stats-collector]
node1_1    | [2016-06-30 09:35:39,050][DEBUG][marvel.agent.collector.shards] [node1] starting collector [shards-collector]
node1_1    | [2016-06-30 09:35:39,050][DEBUG][marvel.agent.collector.indices] [node1] starting collector [index-recovery-collector]
node1_1    | [2016-06-30 09:35:39,050][DEBUG][marvel.agent.collector.indices] [node1] starting collector [indices-stats-collector]
node1_1    | [2016-06-30 09:35:39,050][DEBUG][marvel.agent.collector.indices] [node1] starting collector [index-stats-collector]
node1_1    | [2016-06-30 09:35:39,050][DEBUG][marvel.agent.collector.cluster] [node1] starting collector [cluster-state-collector]
node1_1    | [2016-06-30 09:35:39,057][DEBUG][marvel.cleaner           ] [node1] starting cleaning service
node1_1    | [2016-06-30 09:35:39,067][DEBUG][marvel.cleaner           ] [node1] cleaning service started
node1_1    | [2016-06-30 09:35:39,224][INFO ][transport                ] [node1] publish_address {172.21.0.4:9300}, bound_addresses {[::]:9300}
node1_1    | [2016-06-30 09:35:39,255][INFO ][discovery                ] [node1] elasticsearch/RQZm87mlRsagl_DJv6u1Zw
kibana_1   | {"type":"log","@timestamp":"2016-06-30T09:35:39+00:00","tags":["warning","elasticsearch"],"pid":9,"message":"Unable to revive connection: http://balance:9200/"}
kibana_1   | {"type":"log","@timestamp":"2016-06-30T09:35:39+00:00","tags":["warning","elasticsearch"],"pid":9,"message":"No living connections"}
kibana_1   | {"type":"response","@timestamp":"2016-06-30T09:35:39+00:00","tags":[],"pid":9,"method":"post","statusCode":400,"req":{"url":"/api/marvel/v1/clusters","method":"post","headers":{"host":"localhost:5601","accept":"application/json, text/plain, */*","kbn-version":"4.5.1","accept-language":"en-us","accept-encoding":"gzip, deflate","content-type":"application/json;charset=UTF-8","origin":"http://localhost:5601","content-length":"81","connection":"keep-alive","user-agent":"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_5) AppleWebKit/601.6.17 (KHTML, like Gecko) Version/9.1.1 Safari/601.6.17","referer":"http://localhost:5601/app/marvel","cookie":"__hstc=181257784.6604af83db87ade57e8bf98174f3e534.1463406629839.1463476931487.1463560951397.8; hsfirstvisit=http%3A%2F%2Flocalhost%3A9001%2Flogin.htm%3FredirectTo%3D%2Fapp.htm||1463406629838; hubspotutk=6604af83db87ade57e8bf98174f3e534"},"remoteAddress":"172.21.0.1","userAgent":"172.21.0.1","referer":"http://localhost:5601/app/marvel"},"res":{"statusCode":400,"responseTime":36,"contentLength":9},"message":"POST /api/marvel/v1/clusters 400 36ms - 9.0B"}
kibana_1   | {"type":"log","@timestamp":"2016-06-30T09:35:39+00:00","tags":["warning","elasticsearch"],"pid":9,"message":"Unable to revive connection: http://balance:9200/"}
kibana_1   | {"type":"log","@timestamp":"2016-06-30T09:35:39+00:00","tags":["warning","elasticsearch"],"pid":9,"message":"No living connections"}
kibana_1   | {"type":"response","@timestamp":"2016-06-30T09:35:39+00:00","tags":[],"pid":9,"method":"post","statusCode":400,"req":{"url":"/api/marvel/v1/clusters/xktTAcE9R4y9zN3demV6PA","method":"post","headers":{"host":"localhost:5601","accept":"application/json, text/plain, */*","kbn-version":"4.5.1","accept-language":"en-us","accept-encoding":"gzip, deflate","content-type":"application/json;charset=UTF-8","origin":"http://localhost:5601","content-length":"200","connection":"keep-alive","user-agent":"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_5) AppleWebKit/601.6.17 (KHTML, like Gecko) Version/9.1.1 Safari/601.6.17","referer":"http://localhost:5601/app/marvel","cookie":"__hstc=181257784.6604af83db87ade57e8bf98174f3e534.1463406629839.1463476931487.1463560951397.8; hsfirstvisit=http%3A%2F%2Flocalhost%3A9001%2Flogin.htm%3FredirectTo%3D%2Fapp.htm||1463406629838; hubspotutk=6604af83db87ade57e8bf98174f3e534"},"remoteAddress":"172.21.0.1","userAgent":"172.21.0.1","referer":"http://localhost:5601/app/marvel"},"res":{"statusCode":400,"responseTime":75,"contentLength":9},"message":"POST /api/marvel/v1/clusters/xktTAcE9R4y9zN3demV6PA 400 75ms - 9.0B"}
balance_1  | [2016-06-30 09:35:39,616][INFO ][node                     ] [balance] initialized
balance_1  | [2016-06-30 09:35:39,616][INFO ][node                     ] [balance] starting ...
balance_1  | [2016-06-30 09:35:39,619][DEBUG][marvel.agent.collector.indices] [balance] starting collector [indices-stats-collector]
balance_1  | [2016-06-30 09:35:39,619][DEBUG][marvel.agent.collector.shards] [balance] starting collector [shards-collector]
balance_1  | [2016-06-30 09:35:39,620][DEBUG][marvel.agent.collector.indices] [balance] starting collector [index-recovery-collector]
balance_1  | [2016-06-30 09:35:39,620][DEBUG][marvel.agent.collector.node] [balance] starting collector [node-stats-collector]
balance_1  | [2016-06-30 09:35:39,620][DEBUG][marvel.agent.collector.cluster] [balance] starting collector [cluster-stats-collector]
balance_1  | [2016-06-30 09:35:39,621][DEBUG][marvel.agent.collector.indices] [balance] starting collector [index-stats-collector]
balance_1  | [2016-06-30 09:35:39,621][DEBUG][marvel.agent.collector.cluster] [balance] starting collector [cluster-state-collector]
balance_1  | [2016-06-30 09:35:39,634][DEBUG][marvel.cleaner           ] [balance] starting cleaning service
balance_1  | [2016-06-30 09:35:39,642][DEBUG][marvel.cleaner           ] [balance] cleaning service started
balance_1  | [2016-06-30 09:35:39,765][INFO ][transport                ] [balance] publish_address {172.21.0.6:9300}, bound_addresses {[::]:9300}
balance_1  | [2016-06-30 09:35:39,770][INFO ][discovery                ] [balance] elasticsearch/tflORxopRJiwUslyNG6IaA
node2_1    | [2016-06-30 09:35:40,132][INFO ][cluster.service          ] [node2] new_master {node2}{NlRMBdL1TYSF0049AwX5Vg}{172.21.0.2}{172.21.0.2:9300}, reason: zen-disco-join(elected_as_master, [0] joins received)
node2_1    | [2016-06-30 09:35:40,154][DEBUG][marvel.agent.exporter.local] local exporter [default_local] - waiting until gateway has recovered from disk
kibana_1   | {"type":"log","@timestamp":"2016-06-30T09:35:40+00:00","tags":["warning","elasticsearch"],"pid":9,"message":"Unable to revive connection: http://balance:9200/"}
kibana_1   | {"type":"log","@timestamp":"2016-06-30T09:35:40+00:00","tags":["warning","elasticsearch"],"pid":9,"message":"No living connections"}
node2_1    | [2016-06-30 09:35:40,184][DEBUG][marvel.agent.exporter.local] local exporter [default_local] - could not find existing marvel template for timestamped indices, installing a new one
node2_1    | [2016-06-30 09:35:40,186][DEBUG][marvel.agent.exporter.local] local exporter [default_local] - installing template [.marvel-es-1]
node2_1    | [2016-06-30 09:35:40,202][INFO ][http                     ] [node2] publish_address {172.21.0.2:9200}, bound_addresses {[::]:9200}
node2_1    | [2016-06-30 09:35:40,202][INFO ][node                     ] [node2] started
node2_1    | [2016-06-30 09:35:40,283][INFO ][gateway                  ] [node2] recovered [0] indices into cluster_state
node2_1    | [2016-06-30 09:35:40,285][DEBUG][marvel.agent.exporter.local] found index template [[cursor, index: 5, value: .marvel-es-1]] in version [1]
node2_1    | [2016-06-30 09:35:40,286][DEBUG][marvel.agent.exporter.local] local exporter [default_local] - could not find existing marvel template for data index, installing a new one
node2_1    | [2016-06-30 09:35:40,287][DEBUG][marvel.agent.exporter.local] local exporter [default_local] - installing template [.marvel-es-data-1]
node2_1    | [2016-06-30 09:35:40,312][DEBUG][marvel.agent.exporter.local] found index template [[cursor, index: 5, value: .marvel-es-1]] in version [1]
node2_1    | [2016-06-30 09:35:40,320][DEBUG][marvel.agent.exporter.local] found index template [[cursor, index: 0, value: .marvel-es-data-1]] in version [1]
node2_1    | [2016-06-30 09:35:40,322][DEBUG][marvel.agent.exporter.local] local exporter [default_local] - started!
node2_1    | [2016-06-30 09:35:40,439][DEBUG][marvel.agent.exporter.local] found index template [[cursor, index: 5, value: .marvel-es-1]] in version [1]
node2_1    | [2016-06-30 09:35:40,439][DEBUG][marvel.agent.exporter.local] found index template [[cursor, index: 0, value: .marvel-es-data-1]] in version [1]
node2_1    | [2016-06-30 09:35:40,442][INFO ][license.plugin.core      ] [node2] license [4f78edca-24b8-4844-ab97-97952f4033b4] - valid
node2_1    | [2016-06-30 09:35:40,448][ERROR][license.plugin.core      ] [node2]
node2_1    | #
node2_1    | # License will expire on [Saturday, July 30, 2016]. If you have a new license, please update it.
node2_1    | # Otherwise, please reach out to your support contact.
node2_1    | #
node2_1    | # Commercial plugins operate with reduced functionality on license expiration:
node2_1    | # - marvel
node2_1    | #  - The agent will stop collecting cluster and indices metrics
node2_1    | #  - The agent will stop automatically cleaning indices older than [marvel.history.duration]
node2_1    | [2016-06-30 09:35:40,462][INFO ][cluster.service          ] [node2] added {{node3}{aZC7TyMjTDSK1Iq-LnLYkA}{172.21.0.3}{172.21.0.3:9300},}, reason: zen-disco-join(join from node[{node3}{aZC7TyMjTDSK1Iq-LnLYkA}{172.21.0.3}{172.21.0.3:9300}])
node3_1    | [2016-06-30 09:35:40,527][INFO ][cluster.service          ] [node3] detected_master {node2}{NlRMBdL1TYSF0049AwX5Vg}{172.21.0.2}{172.21.0.2:9300}, added {{node2}{NlRMBdL1TYSF0049AwX5Vg}{172.21.0.2}{172.21.0.2:9300},}, reason: zen-disco-receive(from master [{node2}{NlRMBdL1TYSF0049AwX5Vg}{172.21.0.2}{172.21.0.2:9300}])
node3_1    | [2016-06-30 09:35:40,570][DEBUG][marvel.agent.exporter.local] found index template [[cursor, index: 3, value: .marvel-es-1]] in version [1]
node3_1    | [2016-06-30 09:35:40,584][DEBUG][marvel.agent.exporter.local] found index template [[cursor, index: 6, value: .marvel-es-data-1]] in version [1]
node3_1    | [2016-06-30 09:35:40,586][DEBUG][marvel.agent.exporter.local] local exporter [default_local] - started!
node3_1    | [2016-06-30 09:35:40,726][INFO ][license.plugin.core      ] [node3] license [4f78edca-24b8-4844-ab97-97952f4033b4] - valid
node3_1    | [2016-06-30 09:35:40,735][ERROR][license.plugin.core      ] [node3]
node3_1    | #
node3_1    | # License will expire on [Saturday, July 30, 2016]. If you have a new license, please update it.
node3_1    | # Otherwise, please reach out to your support contact.
node3_1    | #
node3_1    | # Commercial plugins operate with reduced functionality on license expiration:
node3_1    | # - marvel
node3_1    | #  - The agent will stop collecting cluster and indices metrics
node3_1    | #  - The agent will stop automatically cleaning indices older than [marvel.history.duration]
node2_1    | [2016-06-30 09:35:40,812][DEBUG][marvel.agent.exporter.local] found index template [[cursor, index: 5, value: .marvel-es-1]] in version [1]
node2_1    | [2016-06-30 09:35:40,816][DEBUG][marvel.agent.exporter.local] found index template [[cursor, index: 0, value: .marvel-es-data-1]] in version [1]
node3_1    | [2016-06-30 09:35:40,843][INFO ][http                     ] [node3] publish_address {172.21.0.3:9200}, bound_addresses {[::]:9200}
node3_1    | [2016-06-30 09:35:40,844][INFO ][node                     ] [node3] started
node2_1    | [2016-06-30 09:35:42,475][INFO ][cluster.service          ] [node2] added {{node1}{RQZm87mlRsagl_DJv6u1Zw}{172.21.0.4}{172.21.0.4:9300},}, reason: zen-disco-join(join from node[{node1}{RQZm87mlRsagl_DJv6u1Zw}{172.21.0.4}{172.21.0.4:9300}])
node3_1    | [2016-06-30 09:35:42,492][INFO ][cluster.service          ] [node3] added {{node1}{RQZm87mlRsagl_DJv6u1Zw}{172.21.0.4}{172.21.0.4:9300},}, reason: zen-disco-receive(from master [{node2}{NlRMBdL1TYSF0049AwX5Vg}{172.21.0.2}{172.21.0.2:9300}])
node3_1    | [2016-06-30 09:35:42,504][DEBUG][marvel.agent.exporter.local] found index template [[cursor, index: 3, value: .marvel-es-1]] in version [1]
node3_1    | [2016-06-30 09:35:42,505][DEBUG][marvel.agent.exporter.local] found index template [[cursor, index: 6, value: .marvel-es-data-1]] in version [1]
node1_1    | [2016-06-30 09:35:42,508][INFO ][cluster.service          ] [node1] detected_master {node2}{NlRMBdL1TYSF0049AwX5Vg}{172.21.0.2}{172.21.0.2:9300}, added {{node2}{NlRMBdL1TYSF0049AwX5Vg}{172.21.0.2}{172.21.0.2:9300},{node3}{aZC7TyMjTDSK1Iq-LnLYkA}{172.21.0.3}{172.21.0.3:9300},}, reason: zen-disco-receive(from master [{node2}{NlRMBdL1TYSF0049AwX5Vg}{172.21.0.2}{172.21.0.2:9300}])
node1_1    | [2016-06-30 09:35:42,543][DEBUG][marvel.agent.exporter.local] found index template [[cursor, index: 7, value: .marvel-es-1]] in version [1]
node1_1    | [2016-06-30 09:35:42,548][DEBUG][marvel.agent.exporter.local] found index template [[cursor, index: 0, value: .marvel-es-data-1]] in version [1]
node1_1    | [2016-06-30 09:35:42,551][DEBUG][marvel.agent.exporter.local] local exporter [default_local] - started!
node1_1    | [2016-06-30 09:35:42,626][INFO ][license.plugin.core      ] [node1] license [4f78edca-24b8-4844-ab97-97952f4033b4] - valid
node1_1    | [2016-06-30 09:35:42,629][ERROR][license.plugin.core      ] [node1]
node1_1    | #
node1_1    | # License will expire on [Saturday, July 30, 2016]. If you have a new license, please update it.
node1_1    | # Otherwise, please reach out to your support contact.
node1_1    | #
node1_1    | # Commercial plugins operate with reduced functionality on license expiration:
node1_1    | # - marvel
node1_1    | #  - The agent will stop collecting cluster and indices metrics
node1_1    | #  - The agent will stop automatically cleaning indices older than [marvel.history.duration]
node2_1    | [2016-06-30 09:35:42,662][DEBUG][marvel.agent.exporter.local] found index template [[cursor, index: 5, value: .marvel-es-1]] in version [1]
node2_1    | [2016-06-30 09:35:42,662][DEBUG][marvel.agent.exporter.local] found index template [[cursor, index: 0, value: .marvel-es-data-1]] in version [1]
node1_1    | [2016-06-30 09:35:42,678][INFO ][http                     ] [node1] publish_address {172.21.0.4:9200}, bound_addresses {[::]:9200}
node1_1    | [2016-06-30 09:35:42,679][INFO ][node                     ] [node1] started
kibana_1   | {"type":"log","@timestamp":"2016-06-30T09:35:42+00:00","tags":["warning","elasticsearch"],"pid":9,"message":"Unable to revive connection: http://balance:9200/"}
kibana_1   | {"type":"log","@timestamp":"2016-06-30T09:35:42+00:00","tags":["warning","elasticsearch"],"pid":9,"message":"No living connections"}
node2_1    | [2016-06-30 09:35:42,894][INFO ][cluster.service          ] [node2] added {{balance}{tflORxopRJiwUslyNG6IaA}{172.21.0.6}{172.21.0.6:9300}{data=false, master=false},}, reason: zen-disco-join(join from node[{balance}{tflORxopRJiwUslyNG6IaA}{172.21.0.6}{172.21.0.6:9300}{data=false, master=false}])
node3_1    | [2016-06-30 09:35:42,897][INFO ][cluster.service          ] [node3] added {{balance}{tflORxopRJiwUslyNG6IaA}{172.21.0.6}{172.21.0.6:9300}{data=false, master=false},}, reason: zen-disco-receive(from master [{node2}{NlRMBdL1TYSF0049AwX5Vg}{172.21.0.2}{172.21.0.2:9300}])
node3_1    | [2016-06-30 09:35:42,927][DEBUG][marvel.agent.exporter.local] found index template [[cursor, index: 3, value: .marvel-es-1]] in version [1]
node3_1    | [2016-06-30 09:35:42,930][DEBUG][marvel.agent.exporter.local] found index template [[cursor, index: 6, value: .marvel-es-data-1]] in version [1]
node1_1    | [2016-06-30 09:35:42,943][INFO ][cluster.service          ] [node1] added {{balance}{tflORxopRJiwUslyNG6IaA}{172.21.0.6}{172.21.0.6:9300}{data=false, master=false},}, reason: zen-disco-receive(from master [{node2}{NlRMBdL1TYSF0049AwX5Vg}{172.21.0.2}{172.21.0.2:9300}])
node1_1    | [2016-06-30 09:35:42,961][DEBUG][marvel.agent.exporter.local] found index template [[cursor, index: 7, value: .marvel-es-1]] in version [1]
node1_1    | [2016-06-30 09:35:42,961][DEBUG][marvel.agent.exporter.local] found index template [[cursor, index: 0, value: .marvel-es-data-1]] in version [1]
balance_1  | [2016-06-30 09:35:42,970][INFO ][cluster.service          ] [balance] detected_master {node2}{NlRMBdL1TYSF0049AwX5Vg}{172.21.0.2}{172.21.0.2:9300}, added {{node3}{aZC7TyMjTDSK1Iq-LnLYkA}{172.21.0.3}{172.21.0.3:9300},{node1}{RQZm87mlRsagl_DJv6u1Zw}{172.21.0.4}{172.21.0.4:9300},{node2}{NlRMBdL1TYSF0049AwX5Vg}{172.21.0.2}{172.21.0.2:9300},}, reason: zen-disco-receive(from master [{node2}{NlRMBdL1TYSF0049AwX5Vg}{172.21.0.2}{172.21.0.2:9300}])
balance_1  | [2016-06-30 09:35:43,021][DEBUG][marvel.agent.exporter.local] found index template [[cursor, index: 6, value: .marvel-es-1]] in version [1]
balance_1  | [2016-06-30 09:35:43,021][DEBUG][marvel.agent.exporter.local] local exporter [default_local] - started!
balance_1  | [2016-06-30 09:35:43,024][DEBUG][marvel.agent.exporter.local] local exporter [default_local] - started!
balance_1  | [2016-06-30 09:35:43,139][INFO ][license.plugin.core      ] [balance] license [4f78edca-24b8-4844-ab97-97952f4033b4] - valid
balance_1  | [2016-06-30 09:35:43,148][ERROR][license.plugin.core      ] [balance]
balance_1  | #
balance_1  | # License will expire on [Saturday, July 30, 2016]. If you have a new license, please update it.
balance_1  | # Otherwise, please reach out to your support contact.
balance_1  | #
balance_1  | # Commercial plugins operate with reduced functionality on license expiration:
balance_1  | # - marvel
balance_1  | #  - The agent will stop collecting cluster and indices metrics
balance_1  | #  - The agent will stop automatically cleaning indices older than [marvel.history.duration]
node2_1    | [2016-06-30 09:35:43,154][DEBUG][marvel.agent.exporter.local] found index template [[cursor, index: 5, value: .marvel-es-1]] in version [1]
node2_1    | [2016-06-30 09:35:43,154][DEBUG][marvel.agent.exporter.local] found index template [[cursor, index: 0, value: .marvel-es-data-1]] in version [1]
balance_1  | [2016-06-30 09:35:43,172][INFO ][http                     ] [balance] publish_address {172.21.0.6:9200}, bound_addresses {[::]:9200}
balance_1  | [2016-06-30 09:35:43,173][INFO ][node                     ] [balance] started
```

One you have stood up the cluster and populated it with data, it's easy to bring down some of the nodes and see the effect in Kibana's Mravel plugin:

* If you have the stack running from previous, stop it with `ctrl-c`.
* Start it up again, this time using `docker-compose up -d`, this use daemon mode.
* Wait for about 30 secs for it to startup properly.
* Click on the `Indicies` tab at the top of the page and from the list at the bottom choose the item labellled similarly to `.marvel-es-1-2016.06.30`.  This is the Marvel monitoring index.  It has 1 Shard and 1 Replica.
* At the bottom of the next screen is the nodes and the state of the shard and replica (note the color diff and node containing the _primary_ shard.
* Back to the console and shutdown the node conatining the primary shard `docker-compose stop <primary-shard-node-name>`
* In Kibana you should see the primary node disapear and the status will go yellow
* The remaining node containing the replica will be elected as the primary, and in about 30 seconds the a new node will be initialised to hold the required replica and the cluster state should become green again.

Try taking down combinations of node1-3 and observe the results.

To stop the stack, use

```
docker-compose stop
```

This will stop the containers, but not remove their definitions, networks or volumes which contain the data.  You can easily bring the stack back up again with

```
docker-compose start
```

To remove the containers, volumes, networks (but not the images which were built)

```
docker-compose down -v
```

To remove everything including the images

```
docker-compose down -v --rmi all
```






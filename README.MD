Zabbix-Event-EFK
----------------

An example of stack with zabbix and EFK (ElasticSearch, Fluentd, Kibana) configured to use the new realtime event(history,trends,problems) generation that comes
with zabbix 4.0.0+ and ingest it into the ElasticSearch.

https://www.zabbix.com/documentation/4.0/manual/appendix/install/real_time_export

It leverages zabbix componentes available as images from `github.com/gcavalcante8808/zabbix-docker` as well docker and docker-compose.

Requirements
------------

To use the following repo, you need to have the following software installed and/or configured:

 * Docker installed and running and docker-compose installed in the PATH;
 * Git

You still need root access to change the event-data folder ownership (required by zabbix to write events).

Setup
-----

Tipically, you need to:

1. Clone the repository into your machine;
2. Access the new folder created for the project;
3. Create and define permissions to the zabbix event folder;
4. Start the stack with `docker-compose up -d`.

The commands to execute the steps above are:

```
    git clone https://github.com/gcavalcante8808/zabbix-evt-efk.git
    cd zabbix-evt-efk
    mkdir event-data
    chown -R 1000:100 event-data
    docker-compose up -d
```

After all images has been download, check the entire stack state through the command `docker-compose ps`.

View and Explore Data
---------------------

The default viewer is Kibana and its listen on port 5601 by default. Use your browser to access the address `localhost:5601`.

You'll need to define an index to be visualized in kibana before starting to view and explore your data; ES guys have an official guide available in
https://www.elastic.co/guide/en/kibana/current/tutorial-define-index.html to help with that :D

By default, fluentd keeps tracks of all events exported by zabbix (history, trends, problems/events), using the following log_names to insert the entries into elastic:

 * zabbix-event-history
 * zabbix-event-problems
 * zabbix-event-trends

You can disable the ingest of some of them by editing the file `fluentd\fluent.conf` and then, restarting fluentd with `docker-compose restart fluentd`.

Note 1: ElasticSearch Keeps Restarting
--------------------------------------

The elastic search needs the kernel parameter `vm.max_map_count` to be at least 262144 and will refuse to start otherwise.

To fix this, you can use the command `sysctl -w vm.max_map_count=262144`, but it will be lost after next boot.

If you want to make this change permanent, use the following command (as root):

```
    echo "vm.max_map_count=262144" >> /etc/sysctl.d/99-es.conf
```

Note 2: Zabbix Server Keeps Restarting
--------------------------------------

Check the logs of the server, but if this is a fresh install, it probably related with event-data dir permissions. Execute the following commands to fix it (Inside the cloned folder):

```
   chown -R 1000:1000 event-data
   docker-compose restart server
```

Author
------

Author: Gabriel Abdalla Cavalcante Silva (gabriel.cavalcante88@gmail.com)

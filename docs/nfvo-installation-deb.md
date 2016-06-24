## Install NFVO

The NFVO is implemented in java using the [spring.io][spring] framework. For more details about the NFVO architecture, you can refer to the extend it section.

### Install the latest NFVO version from the debian package

The NFVO can be installed using different mechanisms. In this HowTo we will show you how to install it directly from its debian package.

The NFVO uses the AMQP protocol for communicating with the VNFMs. Therefore an implementation of it is necessary, we chose [RabbitMQ][reference-to-rabbit-site].
To facilitate the installation procedures we provide an installation procedure which will install the NFVO and the prerequired libraries.
To execute the following command you need to have curl installed (see http://curl.haxx.se/).

To install the OpenBaton NFVO through its debian package you can type the following command:

```bash
bash <(curl -fsSkL http://get.openbaton.org/bootstrap-deb)
```

At the end of the installation procedure, if there are no errors, the dashboard is reachable at: [localhost:8080] and you should have the following structure:
```bash
/usr/lib/openbaton
├── openbaton-*.jar
├── gvnfm
└── plugins
```

Where:

* `openbaton-*jar` is the jar file related to the version of OpenBaton NFVO which has been installed
* `gvnfm` (present only if during the installation procedure you also installed the Generic VNFM) contains the jar file related to the OpenBaton Generic VNFM
* `plugins` contains the plugins for OpenBaton. So far, if you downloaded the VIM-Driver Plugins during the installation procedure, it will contain only the jar files related to the plugins downloaded

Additionally you should also have the following structure:
```bash
/usr/bin
├── openbaton-nfvo
└── openbaton-gvnfm
```

Where:

* `openbaton-nfvo` is the OpenBaton NFVO executable
* `openbaton-gvnfm` (present only if you also installed the Generic VNFM) is the OpenBaton Generic GVNFM executable

At this point the NFVO is ready to be used. Please refer to the [Introduction][use-openbaton] on how to start using it.

### Starting and stopping NFVO (and the Generic VNFM)

After the installation procedure the NFVO is running. If you want to stop it, enter one of the following commands:
```bash
sudo service openbaton-nfvo stop
sudo stop openbaton-nfvo
sudo openbaton-nfvo stop

```

To start the NFVO, instead, enter one of the following commands:
```bash
sudo service openbaton-nfvo start
sudo start openbaton-nfvo
sudo openbaton-nfvo start
```

**Note (in case you also installed the Generic VNFM):** If you also installed the Generic VNFM it is also already running at the end of the installation procedure. You can stop it with one of the following commands:
```bash
sudo service openbaton-gvnfm stop
sudo stop openbaton-gvnfm
sudo openbaton-gvnfm stop
```

**Note (in case you also installed the Generic VNFM):** You can start the Generic VNFM with one of the following commands:
```bash
sudo service openbaton-gvnfm start
sudo start openbaton-gvnfm
sudo openbaton-gvnfm start
```


### NFVO properties overview

The NFVO is configured with default configuration parameters at the beginning. The configuration file is located at: 
```bash
/etc/openbaton/openbaton.properties
```

This file can be modified for specific parameters. For instance, you can decide to change logging levels (TRACE, DEBUG, INFO, WARN, and ERROR) and mechanisms:
```properties
logging.level.org.springframework=INFO
logging.level.org.hibernate=INFO
logging.level.org.apache=INFO
logging.level.org.jclouds=WARN
# logging.level.org.springframework.security=WARN
logging.level.org.springframework.web = WARN

# Level for loggers on classes inside the root package "org.project.openbaton" (and its
# sub-packages)
logging.level.org.openbaton=INFO

# Direct log to a log file
logging.file=/var/log/openbaton.log
```
Or parameters related with persistency (hibernate):
```properties
# DB properties
spring.datasource.username=admin
spring.datasource.password=changeme
# hsql jdbc
spring.datasource.url=jdbc:hsqldb:file:/tmp/openbaton/openbaton.hsdb
spring.datasource.driver-class-name=org.hsqldb.jdbc.JDBCDriver
spring.jpa.database-platform=org.hibernate.dialect.HSQLDialect
# mysql jdbc
#spring.datasource.url=jdbc:mysql://localhost:3306/openbaton
#spring.datasource.driver-class-name=com.mysql.jdbc.Driver
#spring.jpa.database-platform=org.hibernate.dialect.MySQLDialect
# hibernate properties
spring.jpa.show-sql=false
spring.jpa.hibernate.ddl-auto=create-drop
```

**IMPORTANT NOTES:**

(Keep in mind that whenever some of the parameters below referred are changed, you will need to restart the NFVO)

1) By default RabbitMQ is installed on the host of the NFVO. Be aware of the fact that if you want your VNFM to be executed on a different host, you will need RabbitMQ to be reachable also from the outside.  
So when you want to deploy a VNF (EMS) in a VM which runs on a different host in respect to the NFVO, you will need to configure the rabbitmq endpoint (**nfvo.rabbit.brokerIp**) with the real IP of the NFVO host (instead of localhost).

This can be done changing the following properties of the _/etc/openbaton/openbaton.properties_ file:
```properties
nfvo.rabbit.brokerIp = localhost 
```
to:
```properties
nfvo.rabbit.brokerIp = the rabbitmq broker ip
``` 

2) At the end of the installation the NFVO is working with a in-memory database. In order to start using persistency through mysql database, you need to make the properties changes shown below:
```properties
# DB properties
spring.datasource.username=admin
spring.datasource.password=changeme
# hsql jdbc
# spring.datasource.url=jdbc:hsqldb:file:/tmp/openbaton/openbaton.hsdb
# spring.datasource.driver-class-name=org.hsqldb.jdbc.JDBCDriver
# spring.jpa.database-platform=org.hibernate.dialect.HSQLDialect

# mysql jdbc
spring.datasource.url=jdbc:mysql://localhost:3306/openbaton
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
spring.jpa.database-platform=org.hibernate.dialect.MySQLDialect

# hibernate properties
spring.jpa.show-sql=false
# spring.jpa.hibernate.ddl-auto=create-drop
spring.jpa.hibernate.ddl-auto=update
```
Where:
* _spring.datasource.username_ and _spring.datasource.password_ need to be adapted to the mysql username and password.
* _spring.jpa.hibernate.ddl-auto_ has to be set to **update** if you want the NFVO not to drop all the tables after being shut down and to make it reuse the same tables after restarting.

For more details please see the [Spring Documentation](http://docs.spring.io/spring-boot/docs/current/reference/html/common-application-properties.html) regarding the configuration parameters.

These are other parameters about the configuration of Rabbit MQ:
```properties
#########################################
############## RabbitMQ #################
#########################################

# Comma-separated list of addresses to which the client should connect to.
# spring.rabbitmq.addresses=${nfvo.rabbit.brokerIp}
# Create an AmqpAdmin bean.
spring.rabbitmq.dynamic=true
# RabbitMQ host.
spring.rabbitmq.host= ${nfvo.rabbit.brokerIp}
# spring.rabbitmq.host= localhost
# Acknowledge mode of container.
# spring.rabbitmq.listener.acknowledge-mode=
# Start the container automatically on startup.
# spring.rabbitmq.listener.auto-startup=true
# Minimum number of consumers.
spring.rabbitmq.listener.concurrency= 5
# Maximum number of consumers.
spring.rabbitmq.listener.max-concurrency= 30
# Number of messages to be handled in a single request. It should be greater than or equal to the transaction size (if used).
# spring.rabbitmq.listener.prefetch=
# Number of messages to be processed in a transaction. For best results it should be less than or equal to the prefetch count.
# spring.rabbitmq.listener.transaction-size=
# Login user to authenticate to the broker.
spring.rabbitmq.username= admin
# Login to authenticate against the broker.
spring.rabbitmq.password= openbaton
# RabbitMQ port.
spring.rabbitmq.port=5672
# Requested heartbeat timeout, in seconds; zero for none.
# spring.rabbitmq.requested-heartbeat=
# Enable SSL support.
# spring.rabbitmq.ssl.enabled=false
# Path to the key store that holds the SSL certificate.
# spring.rabbitmq.ssl.key-store=
# Password used to access the key store.
# spring.rabbitmq.ssl.key-store-password=
# Trust store that holds SSL certificates.
# spring.rabbitmq.ssl.trust-store=
# Password used to access the trust store.
# spring.rabbitmq.ssl.trust-store-password=
# Virtual host to use when connecting to the broker.
# spring.rabbitmq.virtual-host=

```

These parameters represent the maximum file size of the VNF Package which can be uploaded to the NFVO and the total maximum request size
```properties
# filesUpload
multipart.maxFileSize=2046MB
multipart.maxRequestSize=2046MB
```

The following properties are related to the plugin mechanism used for loading VIM and Monitoring instances. The `vim-plugin-installation-dir` is the directory where all the jar files are, which implement the VIM interface (see the [vim plugin documentation][vim_plugin_doc]). The NFVO will load them at runtime.  
```properties
########## plugin install ###############
# directory for the vim driver plugins
plugin-installation-dir = /usr/local/lib/openbaton/plugins
# this is path to which the plugins(for example openstack-plugin) will write the output log through NFVO
nfvo.plugin.log.path = 
```

This property allows the user to delete the Network Service Records no matter in which status are they. Pleas note that in any case it is possible to remove a Network Service Record in _NULL_ state.
```properties
# nfvo behaviour
nfvo.delete.all-status= true
```

**MONITORING:** Openbaton allows the monitoring of the VNFs via Zabbix. If you want to use this feature, install and configure Zabbix server following the guide at this page [Zabbix server configuration][zabbix-server-configuration].
Once the Zabbix server is correctly configured and running, you need only to add following property. 
Every time a new Network Service is instantiated, each VNFC (VM) is automatically registered to Zabbix server.

```properties 
nfvo.monitoring.ip = the Zabbix server ip
```

These are other parameters about the configuration of the nfvo behaviour:
```properties
# True to enable security (username and password to access), default to false
nfvo.security.enabled = true
# Wait for the NSR to be deleted
nfvo.delete.wait = false 
# The broker ip that needs to be reachable from all the components
nfvo.rabbit.brokerIp = the broker ip here 
# Set the queues to be autodeleted after the shut down
nfvo.rabbit.autodelete = true 
# Setting the number of plugin active consumers
nfvo.plugin.active.consumers = 5 
# Setting the number of minimum concurrency of the nfvo receivers
nfvo.rabbit.minConcurrency = 5 
# Setting the number of maximum concurrency of the nfvo receivers
nfvo.rabbit.maxConcurrency = 15
# Setting the management port number of rabbitmq
nfvo.rabbit.management.port = 15672 

# Setting the heartbeat between ems and the broker
nfvo.ems.queue.heartbeat = 60
# Set the ems queues to be autodeleted after the shut down
nfvo.ems.queue.autodelete = true
# Set the ems version to be installed
nfvo.ems.version = 0.15

# Allow infinite quotas during the GRANT_OPERATION
nfvo.vim.drivers.allowInfiniteQuota = false
# Execute the start event sequentially and in order based on the VNFDependencies. This implies the NSD not to have cycling dependencies
nfvo.start.ordered = false
# Avoid doing the GRANT_OPERATION
nfvo.quota.check = true
# if true, deleting the nsd will remove also its vnfd
nfvo.vnfd.cascade.delete = false
# if true, deleting the vnfd will remove also its vnfpackage
vnfd.vnfp.cascade.delete = false
# if true, after deleting a NSR, the nfvo will wait for "nfvo.delete.vnfr.wait.timeout" after that timeout the VMs and VNFR will be deleted anyway from the NFVO
nfvo.delete.vnfr.wait = false
# this timeout is useful only if "nfvo.delete.vnfr.wait" is set to true
nfvo.delete.vnfr.wait.timeout = 60
```

Those properties are needed in case you want to tune a bit the performances of the NFVO. When the VNFMs send a message to the NFVO, there is a pool of threads able to process these messages in parallel. These parameters allows you to change the pool configuration, for more details please check the [spring documentation regarding thread pool executor](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/scheduling/concurrent/ThreadPoolTaskExecutor.html) 
```properties
# Thread pool executor configuration
# for info see http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/scheduling/concurrent/ThreadPoolTaskExecutor.html
nfvo.vmanager.executor.corepoolsize = 20
nfvo.vmanager.executor.maxpoolsize = 30
nfvo.vmanager.executor.queuecapacity = 500
nfvo.vmanager.executor.keepalive = 30
```

### Let's move to the next step

Dependening on the approach used for deploying your VNF, you'll have either to install the generic-VNFM or install and register your own VNFM

[spring]:https://spring.io
[localhost:8080]:http://localhost:8080/
[vim_plugin_doc]:vim-plugin
[use-openbaton]:use.md
[reference-to-rabbit-site]:https://www.rabbitmq.com/
[zabbix-server-configuration]:zabbix-server-configuration.md

<!---
Script for open external links in a new tab
-->
<script type="text/javascript" charset="utf-8">
      // Creating custom :external selector
      $.expr[':'].external = function(obj){
          return !obj.href.match(/^mailto\:/)
                  && (obj.hostname != location.hostname);
      };
      $(function(){
        $('a:external').addClass('external');
        $(".external").attr('target','_blank');
      })
</script>
## Install NFVO

The NFVO is implemented in java using the [spring.io][spring] framework. For more details about the NFVO architecture, you can refer to the extend it section.

### Install the latest NFVO version from the source code

The NFVO can be installed using different mechanisms. In this how to we will show you how to instantiate it using directly from the git repo. 

The NFVO uses the Java Messaging System for communicating with the VNFMs. Therefore it is a prerequisite to have ActiveMQ up and running. To facilitate the installation procedures we provide an installation script which can be used for installing the NFVO and the prerequired libraries. Considering that this script needs to install some system libraries, it is required to execute it as super user. To execute the following command you need to have curl installed (see http://curl.haxx.se/). 

```bash
sudo su -
curl -fsSkL http://get.openbaton.org/bootstrap |bash
```

At the end of the installation procedure, if there are no errors, the dashboard is reachable at: [localhost:8080] and you should have the following structure:
```bash
/opt/openbaton/
├── apache-activemq-5.11.1
├── generic-vnfm
└── nfvo
```

**Note** by default security is enabled, therefore the dashboard will ask you to insert username ('admin') and password ('openbaton').

Where:
  
* `apache-activemq-5.11.1` contains the activeMQ software (it is basically downloaded, extracted and executed)  
* `generic-vnfm`contains the source code and scripts required for dealing with the generic-vnfm  
* `nfvo` contains the source code and scripts of the NFVO

At this point the NFVO is ready to be used. Please refer to the [Introduction][use-openbaton] on how to start using it.

**Note:** considering that OpenBaton is installed as **"root"** user, would be good to change permissions of the installations folders for executing the differnet components as standard user. Here an example:
```bash
sudo chown -R username: /opt/openbaton
sudo chown -R username: /etc/openbaton
```

### Starting and stopping NFVO

After the installation procedure the nfvo is running. If you want to stop it, enter this command:
```bash
cd /opt/openbaton/nfvo
./openbaton.sh stop
```

**Note (in case you are also using the generic-vnfm):** remember to stop also the Generic VNFM with the following command:
```bash
cd /opt/openbaton/generic-vnfm
./generic-vnfm.sh stop
```
To start the nfvo, enter the command:
```bash
cd /opt/openbaton/nfvo
./openbaton.sh start
```
**Note (in case you are also using the generic-vnfm):** remember to start also the Generic VNFM with the following command:
```bash
cd /opt/openbaton/generic-vnfm
./generic-vnfm.sh start
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
logging.level.org.jclouds=WARN
logging.level.org.springframework.security=WARN
# Level for loggers on classes inside the root package "org.project.openbaton" (and its
# sub-packages)
logging.level.org.openbaton.nfvo=INFO
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
By deafault ActiveMQ is installed on the host of the NFVO. Be aware of the fact that if you want your VNFM to be executed on a different host, you will need ActiveMQ to be reachable also from the extern.  
**Note:** when you want to deploy a VNF (EMS) in a VM which runs on a different host in respect to the NFVO, you will need to configure the activemq endpoint (spring.activemq.broker-url) with the real IP of the NFVO host (instead of localhost).
```properties
# activeMQ
spring.activemq.broker-url=tcp://localhost:61616
spring.activemq.user=admin
spring.activemq.password=admin
```

These parameters rapresent the maximum file size of the VNF Package which can be uploaded to the NFVO and the total maximum request size
```properties
# filesUpload
multipart.maxFileSize=2046MB
multipart.maxRequestSize=2046MB
```

The following properties are related to the plugin mechanism used for loading VIM and Monitoring instances. The `vim-plugin-installation-dir` is the directory where all the jar files are, which implement the VIM interface (see the [vim plugin documentation][vim_plugin_doc]). The NFVO will load them at runtime.  
```properties
# plugin install
# the plugins inside that directory will be executed at startup
plugin-installation-dir = ./plugins
```

This properties allows the user to delete the Network Service Records no matter in which status are they. Pleas note that in any case it is possible to remove a Network Service Record in _NULL_ state.
```properties
# nfvo behaviour
delete-on-all-status = false
```

Those properties are needed in case you want to tune a bit the performances of the NFVO. When the VNFMs send a message to the NFVO, there is a pool of threads able to process these messages in parallel. These parameters allows you to change the pool configuration, for more details please check the [spring documentation regarding thread pool executor](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/scheduling/concurrent/ThreadPoolTaskExecutor.html) 
```properties
# Thread pool executor configuration
# for info see http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/scheduling/concurrent/ThreadPoolTaskExecutor.html
vmanager-executor-core-pool-size = 20
vmanager-executor-max-pool-size = 25
vmanager-executor-queue-capacity = 500
vmanager-keep-alive = 30
```

Whenever some of those parameters are changed, you will need to restart the orchestrator.

### Let's move to the next step

Dependening on the approach used for deploying your VNF, you'll have either to install the generic-VNFM or install and register your own VNFM

[spring]:https://spring.io
[localhost:8080]:http://localhost:8080/
[vim_plugin_doc]:vim-plugin
[use-openbaton]:use.md

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
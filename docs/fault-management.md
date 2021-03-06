# Open Baton Fault Management System (FMS)  
The Open Baton FMS project is a module of the Open Baton Orchestrator. 
It provides an extensive Fault Management System that is based on alarms coming from the VIM and executes actions through the NFVO.  
**Note**: if you followed the installation guide using the bootstrap, you can jump to the section "How to use Open Baton FM"

# Technical Requirements

The technical requirements are:  

- Zabbix plugin running (see the [doc of Zabbix plugin][zabbix-plugin-doc])
- Mysql server installed and running
- Open Baton 3.2.x running
- Generic VNFM 3.2.x running

# How to install Open Baton FM

Once the prerequisites are met, you need to execute the following steps.

## Create the database

In order to create the database be sure you have installed [mysql server][mysql-installation-guide] as already mentioned in the requirements section. 
You need root access to mysql-server in order to create a new database called faultmanagement. Once you access into mysql, execute the following operation: 

```bash
create database faultmanagement;
```

Once the database has been created, you should create a user which will be used by the FM system to access and store data on the database. If you decide to use the `root` user you can skip this step, but you need to modify the fms.properties file accordingly as defined in the next section. 
By default username and password are set with the following values in the fms.properties properties file (see next section if you plan to use a different user and password): 

* username=fmsuser
* password=changeme

Grant the access to the database "faultmanagement", to the user, running the following command:

```bash
GRANT ALL PRIVILEGES ON faultmanagement.* TO fmsuser@'%' IDENTIFIED BY 'changeme';
```

## Modify fms.properties file in order to use different credentials for the database 

In the folder "etc" of this project, there is a file called fms.properties containing all the default properties values used by the FM system. 

In order to use different credentials, you need to modify the following DB properties: 

```bash
# DB properties
spring.datasource.username=fmsuser
spring.datasource.password=changeme
```

In case your DB is running remotely, you can specifcy a different host, instead of localhost, in the following property (be careful to have port 3306 open and accessible from remote): 

```bash
spring.datasource.url=jdbc:mysql://localhost:3306/faultmanagement
```

## Additional configuration options 

As already mentioned in the previous section, in the folder "etc" of this project, there is a file called fms.properties containing all the default properties values used by the FM system.
You should update this file in order to make it work with your NFVO instance. Change the Open Baton related properties section: 

```bash
################################################
####### Open Baton Related properties ##########
################################################
nfvo.ip=localhost
nfvo.port=8080
nfvo-usr=admin
nfvo-pwd=openbaton
```


## Checkout the source code of the project, compile and run it

You can clone this repository with this command:

```bash  
git clone https://github.com/openbaton/fm-system.git
```

The configuration file is etc/fms.properties, you have to copy it in the Open Baton etc folder ( /etc/openbaton ). You can do it typing the following command 

```bash  
cd fm-system
cp etc/fms.properties /etc/openbaton/fms.properties
```

Now, you can finally compile and start the FM System. 

```bash  
./fm-system.sh compile start
```

# How to use Open Baton FM

Open Baton FM is a rule-driven tool. The rules define when to generate an alarm and how to react. The rule for generating the alarm is called fault management policy (see the next section). 
The rule for defining how to react upon alarms is a Drools Rule. Once such rules are in place, Open Baton FM follows the following workflow.

![Fault management system use case][fault-management-system-use-case]

The actions are listed below:

| ACTION              | DESCRIPTION     | 
| ------------------- | --------------  | 
| Heal   |  The VNFM executes the scripts in the Heal lifecycle event (in the VNFD). The message contains the cause of the fault, which can be used in the scripts. 
| Switch to stanby VNFC (Stateless)   |  If the VDU requires redoundancy active-passive, there will be a component VNFC* in standby mode. This action consists in: activate the VNFC*, route all signalling and data flow(s) for VNFC to VNFC*, deactivate VNFC.
| Switch to stanby VNFC (Stateful)    |  To investigate. Refer on ETSI GS NFV-REL 001 v1.1.1 (2015-01) Chapter 11.2.1.

## Write a fault management policy

The fault management policy needs to be present in the VNFD, in particular in the VDU. This is an example of fault management policy:

```json
"fault_management_policy":[
    {
      "name":"web server not available",
      "isVNFAlarm": true,
      "criteria":[
      {
        "parameter_ref":"net.tcp.listen[80]",
        "function":"last()",
        "vnfc_selector":"at_least_one",
        "comparison_operator":"=",
        "threshold":"0"
      }
      ],
      "period":5,
      "severity":"CRITICAL"
    }
]
```

You can find an example of the fault management policy for the **TOSCA YAML descriptors** [here][fm-tosca].

Description of the fault management policy:  

| Property              | Description     
| ------------------- | --------------  
| name   |  This is the name of the fault management policy.
| isVNFAlarm   |  True, if the alarm is of type VNF.
| criteria | The criteria defines a threshold on a monitoring parameter. When the threshold is crossed an alarm is fired.
|period | The criteria is checked every "period" seconds.
|severity | Defines the severity of the alarm.

Description of the criteria:  

| Property              | Description     
| ------------------- | --------------  
| parameter_ref | This is the reference to a monitoring parameter in the VDU. (see below how to define monitoring parameters).
| function | The function to apply to the parameter ( last(0) means the last value available of the parameter). Since currently only Zabbix is supported, look at the [Zabbix documentation][zabbix-functions] for the all available functions. 
|vnfc_selector | select if the criteria is met when all VNFC components cross the threshold (all) or at least one (at_least_one)
| comparison_operator | This defines the comparison operator used for the threshold.
|threshold | The value of the threshold to compare against the parameter_ref value.

In order to refer to a monitoring parameter with the property **parameter_ref**, it needs to be present in the vdu:
 
```json
"monitoring_parameter":[
   "agent.ping",

   "net.tcp.listen[5001]",

   "system.cpu.load[all,avg5]",

   "vfs.file.regmatch[/var/log/app.log,Exception]"
]
```
You can specify every parameter available in the [Zabbix Agent][zabbix-agent-items].

## How the HEAL method works

The Heal VNF operation is a method of the VNF lifecycle management interface described in the ETSI [NFV MANO] specifications. Here is reported the description and the notes about this method:

```
Description: this operation is used to request appropriate correction actions in reaction to a failure.
Notes: This assumes operational behaviour for healing actions by VNFM has been described in the VNFD. An example might be switching between active and standby mode.
```

In the ETSI draft "NFV-IFA007v040" at [this][etsi-draft-Or-VNFM] page, the Heal VNF message is defined as:

```
vnfInstanceId  : Identifies the VNF instance requiring a healing action. 
cause : Indicates the reason why a healing procedure is required. 
```

Once the fault management system received an alarm from the VIM, 
it checks, if the alarm is referred to a VNF and sends the Heal VNF message to the NFVO which forwards it to the respective VNFM.
The VNFM executes in the failed VNFC the scripts in the HEAL lifecycle event.
Here an example of the heal script you can use:

```bash
#!/bin/bash

case "$cause" in

("serviceDown") 
	echo "Apache is down, let's try to restart it..."
	service apache2 restart
	if [ $? -ne 0 ]; then
	    echo "ERROR: the Apache service is not started"
	    exit 1
    	fi
	echo "The Apache service is running again!"
	;;
*) echo "The cause $cause is unknown"
	exit 2
	;;
esac
```

The variable $cause is specified in the Drools rule. In our case it is "serviceDown" and we try to restart the Apache server.

## Drools Rules

The Open Baton FMS is a rule-based system. Such rules are specified in Drools language and processed by the Drools engine in the Open Baton FMS.
An example rule looks like the following:
```
rule "Save a VNFAlarm"
    when
        vnfAlarm : VNFAlarm()
    then
    VNFAlarm alarm = vnfAlarmRepository.save(vnfAlarm);
    logger.debug("Saved VnfAlarm: "+alarm);
end
```

This rule saves a VNF Alarm in the database.
The following rule executes the HEAL action once a VNF Alarm is received:

```
rule "Got a critical VNF Alarm and execute the HEAL action"

    when
       vnfAlarm : VNFAlarm(  alarmState == AlarmState.FIRED, perceivedSeverity == PerceivedSeverity.CRITICAL)
    then

    //Get the vnfr
    VirtualNetworkFunctionRecord vnfr = nfvoRequestorWrapper.getVirtualNetworkFunctionRecord(vnfAlarm.getVnfrId());

    //Get the vnfc failed (assuming only one vnfc is failed)
    VNFCInstance vnfcInstance = nsrManager.getVNFCInstanceFromVnfr(vnfr,vnfAlarm.getVnfcIds().iterator().next());

    logger.info("(VNF LAYER) A CRITICAL alarm is received by the vnfc: "+vnfcInstance.getHostname());

    //Get the vdu of the failed VNFC
    VirtualDeploymentUnit vdu = nfvoRequestorWrapper.getVDU(vnfr,vnfcInstance.getId());

    logger.info("Heal fired!");
    highAvailabilityManager.executeHeal("serviceDown",vnfr.getParent_ns_id(),vnfr.getId(),vdu.getId(),vnfcInstance.getId());

    //Insert a new recovery action

    RecoveryAction recoveryAction= new RecoveryAction(RecoveryActionType.HEAL,vnfr.getEndpoint(),"");
    recoveryAction.setStatus(RecoveryActionStatus.IN_PROGRESS);
    insert(recoveryAction);
end
```


## How the Switch to Standby works

The Switch to Standby action can be performed by the Open Baton FMS once a VNFC in standby is present in the VNF. Its main action is switch the service from a VNFC to the VNFC in standby automatically.
In order to have a VNFC in standby, such information must be included in the VNFD, in particular in the VDU, as the following:

```
"high_availability":{
	"resiliencyLevel":"ACTIVE_STANDBY_STATELESS",
	"redundancyScheme":"1:N"
}
```
This information will be processed by the Open Baton FMS which will create a VNFC instance in standby.
Then in a Drools rule this action can be called as following:
```
highAvailabilityManager.switchToRedundantVNFC(failedVnfcInstance,vnfr,vdu);
```

## Tutorial

The tutorial is available at this [page][fms-tuto-page]. 

[openbaton]: http://openbaton.org
[openbaton-doc]: http://openbaton.org/documentation
[openbaton-github]: http://github.org/openbaton
[openbaton-logo]: https://raw.githubusercontent.com/openbaton/openbaton.github.io/master/images/openBaton.png
[openbaton-mail]: mailto:users@openbaton.org
[openbaton-twitter]: https://twitter.com/openbaton
[fms-tuto-page]:fms-sipp-tutorial.md
[fm-tosca]:tosca-fm.md

[zabbix-plugin-doc]:https://github.com/openbaton/docs/blob/develop/docs/zabbix-plugin.md
[create-db]:#create-the-database
[fault-management-system-use-case]:images/fms-use-case.png
[zabbix-functions]:https://www.zabbix.com/documentation/2.2/manual/appendix/triggers/functions
[zabbix-agent-items]:https://www.zabbix.com/documentation/2.2/manual/config/items/itemtypes/zabbix_agent
[NFV MANO]:http://www.etsi.org/deliver/etsi_gs/NFV-MAN/001_099/001/01.01.01_60/gs_nfv-man001v010101p.pdf
[etsi-draft-Or-VNFM]:https://docbox.etsi.org/isg/nfv/open/Drafts/IFA007_Or-Vnfm_ref_point_Spec/
[mysql-installation-guide]:http://dev.mysql.com/doc/refman/5.7/en/linux-installation.html

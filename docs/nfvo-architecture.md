# NFVO Architecture

*NFVO* is a modular software composed by the modules illustrated in the following picture:

![NFVO module architecture][nfvo-architecture-full]

##### API

This module contains the necessary classes exposing APIs as ReST server. For more details please see [the api documentation][api]

##### MAIN

This module contains the classes in charge of the startup of the whole system and gathering configuratons for instance.

##### COMMON

This module contains the classes that are common to the NFVO

##### CLI

This module contains the NFVO console.

##### DASHBOARD

This module contains the web dashboard available at localhost:8080

##### CORE-INT

This module contains the interfaces of the core functionalities regarding only the internal NFVO interfaces. Most of them are definend in [ETSI MANO specification][nfv-mano] in NFV-MANO interfaces section.

##### CORE-IMPL

This module contains the beans implementing the **core-int** interfaces.

##### VNFM-INT

This module contains the interfaces of the core functionalities regarding only the NFVO interfaces to the VnfManagers. Most of them are definend in [ETSI MANO specification][nfv-mano] in NFV-MANO interfaces section.

##### VNFM-IMPL

This module contains the beans implementing the **vnfm-int** interfaces.

##### REPOSITORY

This module contains specific repositories interfacing the database in a generic way using Spring CrudRepository.

##### CATALOGUE

This module contains the complete model of NFVO that is sharde in the openbaton libraries. The model is definend in [ETSI MANO specification][nfv-mano].

##### VIM-INT

This module contains the interfaces of the core functionalities regarding only the NFVO interfaces to the VIM. Most of them are definend in [ETSI MANO specification][nfv-mano] in NFV-MANO interfaces section.

##### VIM-IMPL

This module contains the beans implementing the **vim-int** interfaces.

##### PLUGIN

This module contains the utility classes used to interface to the openbaton plugins.

##### VIM-DRIVERS

This module contains the interface for the VIM openbaton plugins.

##### EXCEPTION

This module contains all the exception classes common to every project containing openbaton libraries.

##### MONITORING

This module contains the interface for the monitoring of the openbaton plugins.

##### TOSCA-PARSER

This module contains all the classes of the parser in charge of managing the TOSCA packages

##### SECURITY

This module configures the security mechanisms used in the NFVO



All the platform modules are based on the [Spring Framework](http://spring.io/) and the comunication between NFVO and other external modules are mostly based on RabbitMQ messaging system.  

<!---
References
-->

[api]: http://get.openbaton.org/api/ApiDoc.pdf
[nfvo-architecture-full]:images/nfvo-architecture-full.png
[nfv-mano]: http://www.etsi.org/deliver/etsi_gs/NFV-MAN/001_099/001/01.01.01_60/gs_NFV-MAN001v010101p.pdf

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
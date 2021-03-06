# TOSCA Iperf Scenario

The definition follows the TOSCA Simple Profile for Network Functions Virtualization (NFV) [Version 1.0][tosca-nfv]. Information about creating a complete TOSCA Template from scratch can be found in this [tutorial][ns-template].

This tutorial explains how to deploy a network service that uses iPerf. [iPerf][iperf-website] is a tool for active measurements of the maximum achievable bandwidth between two or more machines.
This tutorial makes use of:

* Generic VNFM ([generic-vnfm][generic-vnfm])
* Generic EMS ([generic ems][ems-github])
* OpenStack plugin ([openstack-plugin][openstack-plugin])

## Requirements

In order to execute this scenario, you need to have the following components up and running:

 * The NFVO
 * the OpenStack plugin
 * the Generic VNFM (includes already generic EMS)
 * a PoP (of type openstack) registered. You can use the following [json descriptor][vim] by changing the values to your needs.
 
More information about the Iperf Scenario [here][iperf-scenario].

### Complete Example


```yaml
tosca_definitions_version: tosca_simple_profile_for_nfv_1_0
description: Example of NSD

metadata:
  ID: NSD-Iperf+FloatingIps
  vendor: Fokus
  version: 1.0

topology_template:

  node_templates:

    iperf-server: #VNF1
        type: openbaton.type.VNF
        properties:
          vendor: Fokus
          version: 1.0
          endpoint: generic
          type: server
          vnfPackageLocation: https://github.com/openbaton/vnf-scripts.git
          deploymentFlavour:
            - flavour_key: m1.small
        requirements:
          - virtualLink: private
          - vdu: VDU2        
        interfaces:
          lifecycle: # lifecycle
            INSTANTIATE:
              - install.sh
              - install-srv.sh

    iperf-client:
      type: openbaton.type.VNF
      properties:
        ID: x
        vendor: Fokus
        version: 1.0
        type: client
        vnfPackageLocation: https://github.com/openbaton/vnf-scripts.git
        deploymentFlavour:
          - flavour_key: m1.small
        endpoint: generic
      requirements:
         - virtualLink: private
         - vdu: VDU1
      interfaces:
          lifecycle: # lifecycle
            INSTANTIATE:
              - install.sh
            CONFIGURE:
              - server_configure.sh

    VDU1:
      type: tosca.nodes.nfv.VDU
      properties:
        scale_in_out: 1
      artifacts:
        VDU1Image:
          type: tosca.artifacts.Deployment.Image.VM
          file: ubuntu-14.04-server-cloudimg-amd64-disk1

    VDU2:
      type: tosca.nodes.nfv.VDU
      properties:
        scale_in_out: 3
      artifacts:
        VDU1Image:
          type: tosca.artifacts.Deployment.Image.VM
          file: ubuntu-14.04-server-cloudimg-amd64-disk1

    CP1:
      type: tosca.nodes.nfv.CP
      properties:
        floatingIP: random
      requirements:
        - virtualBinding: VDU1
        - virtualLink: private

    CP2: #endpoints of VNF2
      type: tosca.nodes.nfv.CP
      requirements:
        - virtualBinding: VDU2
        - virtualLink: private

    private:
      type: tosca.nodes.nfv.VL
      properties:
        vendor: Fokus

relationships_template:
  connection_server_client: 
    type: tosca.nodes.relationships.ConnectsTo
    source: iperf-server
    target: iperf-client
    parameters:
        - private

```

**NOTE**: Save the definition in a file called iperf-floating-ips.yaml.

## Onboarding

The Iperf Template can be added used in [CSAR][csar] and onboarded using the [Dashboard][dashboard].

<!------------
References
-------------->
[tosca-nfv]: http://docs.oasis-open.org/tosca/tosca-nfv/v1.0/tosca-nfv-v1.0.html
[ETSI-MANO]: https://www.etsi.org/deliver/etsi_gs/NFV-MAN/001_099/001/01.01.01_60/gs_NFV-MAN001v010101p.pdf
[TOSCA-simple-YAML]: http://docs.oasis-open.org/tosca/TOSCA-Simple-Profile-YAML/v1.0/csprd01/TOSCA-Simple-Profile-YAML-v1.0-csprd01.html#_Toc430015766

[iperf-TOSCA]:images/iperf.png

[vnfr-states]:vnfr-states
[vnfm-generic]: vnfm-generic
[nsd-doc]:ns-descriptor
[vnf-package]:vnf-package
[vim-doc]:vim-instance
[dummy-NSR]:dummy-NSR
[iperf-scenario]:iperf-NSR
[cli]: nfvo-how-to-use-cli
[dashboard]: nfvo-how-to-use-gui
[iperf-client-server]:images/use-case-example-iperf-client-server.png
[sequence-diagram-os-vnfm-ems]:images/use-case-example-sequence-diagram-os-vnfm-ems.png
[nsd-doc]:ns-descripto
[iperf]:https://iperf.fr
[generic-vnfm]:https://github.com/openbaton/generic-vnfm
[openstack-plugin]:https://github.com/openbaton/openstack-plugin
[vim]: descriptors/vim-instance/openstack-vim-instance.json
[iperf-nsd-privateIPs]: descriptors/tutorial-iperf-NSR/tutorial-iperf-NSR-privateIPs.json
[iperf-nsd-floatingIPs]: descriptors/tutorial-iperf-NSR/tutorial-iperf-NSR-floatingIPs.json
[nsd-deployment]: images/tutorials/tutorial-iperf-NSR/nsd-deploy.png
[nsd-onboarding]: images/tutorials/tutorial-iperf-NSR/nsd-onboarding.png
[ems-github]: https://github.com/openbaton/ems/tree/master
[iperf-website]:https://iperf.fr
[ns-template]:tosca-nsd
[csar]:tosca-CSAR-onboarding
[dashboard]:nfvo-how-to-use-gui

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





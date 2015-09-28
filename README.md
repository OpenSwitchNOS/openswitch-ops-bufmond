#Read me for ops-bufmond repository
# Table of contents


[TOC]

##What is ops-bufmond?
The primary goal of the buffer monitoring  module to monitor the buffer space consumption inside the switch hardware device. The ops-bufmond module can be managed through any monitor agent application, similar to the broadview project [broadview](https://github.com/Broadcom-Switch/BroadView-Instrumentation).

##What is the structure of the repository?

The buffer monitoring feature changes spread across multiple repository.
* ops-bufmond/ops_bufmond.py python daemon populates the counters information into ovs-db tables described as per hardware description file.
* ops-config-<asic>/bufmond.yaml buffer monitoring counters hardware description file
* ops-openvswitch/vswitchd/bufmon-provider.c bufmon layer (bufmon provider) API's to configure switch hardware


##What is the license?
Apache 2.0 license. For more details refer to COPYING


##What other documents are available?

For the high level design of ops-bufmond, refer to [Buffer Monitoring design - TBL](DESIGN.md)

For general information about OpenSwitch project refer to http://www.openswitch.net
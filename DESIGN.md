#High level design of ops-bufmond (Buffer Monitoring)
#Table of contents
[toc]

Buffers Monitoring (BufMon) is a feature that provides Openvswitch users the ability to monitor buffer space consumption inside the ASIC. It's useful for troubleshooting complex performance problems in the data center networking.

The feature is derived from Broadcom BroadView ([https://github.com/Broadcom-Switch/BroadView-Instrumentation](https://github.com/Broadcom-Switch/BroadView-Instrumentation "Broadview")), however, it is designed to be generic and not tied to specific ASICs. Any ASIC which has buffer counters, can provide this information through this feature interface.

##Reponsibilities

The ops-bufmond module is responsible for

- Monitoring and collecting the buffer space consumption inside the ASIC


## Design choices

The design decisions made for buffer monitoring modules are:

- bufmond:
	-  The bufmond python script is responsible to create counters details into the bufmon table
	- This python script uses the pyyaml library to parse the hardware description file

- Sysd:
	-  The sysd daemon will create the proper symlink to ASIC specific bufmond hardware description yaml file

- Switchd:
  -  The platform independent piece of code handles configuration and statistics collection, based on      buffer monitoring counters configuration residing in the database
  -  The platform dependent bufmon layer (bufmon provider) provides the interface to configure  switch hardware and periodic statistics collection from the switch hardware.

##Relationships to external OpenSwitch entities

The following diagram provides detailed description of relationships and interactions of buffer monitoring modules with other modules in the switch.
```ditaa
	


	                    +-------------------------------------------------------+
	                    |                                                       |
	                    |                            +-----------------------+  |
	                    |                       +-+  | Switchd               |  |
	+------------+      |      +-----------+    |O|  | +------------------+  |  |
	| Broadview  |      |      | Broadview +<-->+V|  | | ASIC Independent |  |  |
	| Collectors +<----------> |  Daemon   |    |S+<-->|   Bufmon.c       |  |  |
	+------------+      |      +-----------+    |D|  | +------------------+  |  |
	                    |      +-----------+    |B|  |                       |  |
	                    |      |   Sysd    +<-->+ |  | +------------------+  |  |
	                    |      |           |    |S|  | |   ASIC Bufmon    |  |  |
	                    |      +-----------+    |E|  | |    Provider      |  |  |
	                    |      +-----------+    |R|  | +------------------+  |  |
	                    |      |  Bufmond  +<-->+V|  |                       |  |
	                    |      |           |    |E|  | +------------------+  |  |
	                    |      +-----------+    |R|  | |    ASIC SDK      |  |  |
	                    |                       +-+  | +------------------+  |  |
	                    |                            +-----------------------+  |
	                    |      +----------------------------------------+       |
	                    |      |                          +-----------+ |       |
	                    |      |                          |ASIC Driver| |       |
	                    |      |                          +-----------+ |       |
	                    |      | Kernel                                 |       |
	                    |      +----------------------------------------+       |
	                    |                                                       |
	                    +-------------------------------------------------------+
```


##OVSDB-Schema

The buffer monitoring related columns on openSwitch table are bufmon\_config column and bufmon\_info column. The "bufmon\_config" column has the configuration information and the "bufmon\_info" has the buffer monitoring capabilities information of the switch hardware. The buffer monitoring counter details realted columns handled on bufmon table.
```ditaa
+------------------------------------------------------------------------------------+
|                                                                                    |
|                                                                   OVSDB            |
|                                                                                    |
|  +----------------------------+       +--------------------------------+           |
|  |                   System   |       |                       bufmon   |           |
|  |                            |       |  hw_unit_id                    |           |
|  |                            |       |  name                          |           |
|  |                            |       |  counter_vendor_specific_info  |           |
|  |                            |       |  enabled                       |           |
|  | bufmon_config              |       |  counter_value                 |           |
|  | bufmon_info                |       |  status                        |           |
|  |                            |       |  trigger_threshold             |           |
|  +----------------------------+       +--------------------------------+           |
|                                                                                    |
+------------------------------------------------------------------------------------+
```

System Table:

The keys and values supported by the bufmon\_config column are:

|    Key                                  |    Value       | Description               |
| ----------------------------------------|----------------|-----------------------------------
| enabled                                 | string         | Specifies  whether the feature is         															  enabled on the system.  False, if the                                                                value is not present.
| counters_mode                           | string         | Specifies whether counters should 															         report their current values or peak                                                                  values since last collection.
| periodic_collection_enabled             | string         | Specifies  whether  periodic                                                                        collection of the counters is enabled.                                                              False, if the value is not present.
| collection_period                       | string         | Specifies the period of collection in                                                                seconds. Five, if the value is not                                                                  present.
| threshold_trigger_collection_enabled    | string         | Specifies whether counters should be                                                                collected immediately when one  of  the                                                              thresholds is crossed. True, if the                                                                  value is not present.
| threshold_trigger_rate_limit            | string         | Specifies maximum number of trigger                                                                  generated  reports  per minute. The                                                                  limit will be averaged and imposed on a                                                              per second basis. For example, value of                                                              600 will allow report every 100ms. If                                                                no value is set, the default is 60 i.e.                                                              once per second.
| snapshot_on_threshold_trigger           | string         | Specifies whether counters should  be                                                                frozen when one of the thresholds is                                                                crossed. True, if the value is not                                                                  present.


The keys and values supported by the bufmon\_info column are:

|    Key                                  |    Value       | Description               |
| ----------------------------------------|----------------|-----------------------------------
| cap_mode_peak                           | string         | Specifies whether the system is capable                                                              of collecting peak  values of the                                                                    counters. False, if the value is not                                                                present.
| cap_mode_current                        | string         | Specifies whether the system is 													                 capable of collecting current values of                                                              the counters. False, if the value is                                                                not present.
| cap_snapshot_on_threshold_trigger       | string         | Specifies whether the system is capable                                                              of freezing counters values on                                                                      threshold crossing. False, if the                                                                    value is not present.
| cap_threshold_trigger_collection        | string         | Specifies whether the system is capable                                                              of immediately collecting values on                                                                  threshold crossing. False, if the value                                                              is not present.
| last_collection_timestamp               | string         | Specifies the timestamp of the last                                                                  collection in seconds from Jan 1, 1970.
Bufmon Table:

|    Column                               | Type           | Description               |
| ----------------------------------------|----------------|-----------------------------------
| hw_unit_id                              | integer        | Identifies the switch hardware unit                                                                  number that counter belongs to.
| name                                    | string         | Name of the counter as it will be shown                                                              in the management systems and will be                                                                referenced by all the interested                                                                    agents. No spaces should be used in the                                                              name.
| counter_vendor_specific_info            | map string     | Contains any information that might                                                                  help ASIC specific driver to identify                                                                the  counter. Both keys and values are                                                              driver and ASIC specific.
| enabled                                 | bool           | Specified whether counter is enabled.
| trigger_threshold                       | integer        | Specified counter specific threshold                                                                limit that would trigger  collection.
| counter_value                           | integer        | Last collected value of the counter.
| status 						          | string         | Specifies the status of the counter.                                                                one of ok, not properly configured, or                                                              triggered


##Internal structure

The various functionality of sub modules are :

####BroadView####
This daemon being contributed by Broadcom and is based on the one provided in Github referenced above. Main difference from the one in GitHub, would be it's communication to OVSDB. Its responsibility is to communicate to the user, over its JSON-RPC protocol and configure/retrieve counters through the database.

####Bufmond####
This daemon is responsible to create one row per counter in the bufmon table according to hardware description file. This daemon also publish the buffer monitoring capabilities information into the bufmon\_info column which is part of Sytem table.

####Sysd####
Sysd creates proper symlink to ASIC specific hardware description file, it would contain a description of all the counters that exist in a given ASIC as well as capabilities of the specific ASIC.

####Switchd####
The switchd configures switch hardware based on buffer monitoring  configuration in the System and bufmon table.Switchd uses the bufmon layer (bufmon provider) API's to configure the switch hardware and for the statistics collection from the switch hardware. In switchd the thread "bufmon_ stats_thread" is responsible to collect statistics periodically from the switch hardware and it will also monitor for the trigger notifications from the switch hardware. The same thread will notify the switchd main thread to push the counter statistics into the database.

####Hardware description file####
The buffer monitoring counters hardware description YAML file (bufmond.yaml) has to be generated per specific platform and not generically for the switch hardware. The reason is that ASICs might provide different sets of counters given different configurations on the specific platform.

Example buffer monitoring capabilities and counters yaml description:

cap_mode_current: true
cap_mode_peak: true
cap_snapshot_on_threshold_trigger: true
cap_threshold_trigger_collection: true
counters:
\- name: device/data/NONE/NONE
  counter_vendor_specific_info:
    counter_name: data
  hw_unit_id: 0
\- name: ingress-port-priority-group/um-share-buffer-count/1/1
  counter_vendor_specific_info:
    counter_name: um-share-buffer-count
    port: '1'
    priority-group: '1'
  hw_unit_id: 0


##References

* [Buffer Monitoring Reference - TBL](http://www.openswitch.net/docs/redest1)
# Buffer Monitoring component test cases

## Contents##
- [Test to verify bufmond without hardware description file](#test-to-verify-bufmond-without-hardware-description-file)
   
- [Test to verify bufmond with hardware description file](#test-to-verify-bufmond-with-hardware-description-file)
    - [Test to verify buffer monitoring counters configuration](#test-to-verify-buffer-monitoring-counters-configuration)
    - [Test to verify periodic polling interval configuration](#test-to-verify-periodic-polling-interval-configuration)
    - [Test to verify trigger based counter collection](#test-to-verify-trigger-based-counter-collection)
    - [Test cases to verify trigger\_rate\_limit settings](#test-cases-to-verify-trigger\_rate\_limit-settings)

##Test cases to verify the Buffer Monitoring Component##
### Objective ###
Test cases to verify that the Buffer Monitoring daemon work.
### Requirements ###
The requirements for this test case are:

 - AS5712 switch
 - Ixia

### Setup ###
#### Topology diagram ####
```ditaa
                                +-----------------------------------+
                                |                                   |
                                |           AS5712 switch           |
                                |                                   |
                                |                                   |
                                |                                   |
                                |   +-----+---+---+----+--------+   |
                                |   |  1  | 2 | 3 | 4  |  ...   |   |
                                |   +---------------------------+   |
                                +-----------------------------------+
                                       |     |
                                       |     +-+
                                       +-+     |
                                         |     |
                                    +-----------------------+
                                    |  +-----------------+  |
                                    |  |  1  | 2 | 3 | 4 |  |
                                    |  +-----+---+---+---+  |
                                    |                       |
                                    |                       |
                                    |                       |
                                    |                       |
                                    |        Ixia           |
                                    +-----------------------+

```

### Test to verify bufmond without hardware description file   ###
#### Description ####
Test to verify that the bufmond daemon without the buffer monitoring counters hardware description file (bufmond.yaml).

### Test result criteria ###
#### Test pass criteria ####
Test case result is a success if the "No rows found in the bufmon table" error message is received and bufmond daemon exits properly.
#### Test fail criteria ####
Test case result is a fail if the "No rows found in the bufmon table" error message is not received.

### Test to verify bufmond with hardware description file  ###
#### Description ####
Test to verify that the bufmond daemon with the buffer monitoring counters hardware description file (bufmond.yaml).


### Test result criteria ###
#### Test pass criteria ####
Test case result is a success if the bufmond is able to detect an bufmond.yaml and able to parse the hardware description file and if all the counters list is added part of the bufmon table.
#### Test fail criteria ####
Test case result is a fail if the bufmond is not able locate the path of the bufmond.yaml or bufmon table in the ovsdb is empty.


### Test to verify buffer monitoring counters configuration   ###
#### Description ####
Test to verify that the buffer monitoring counters configuration option enable/disable.

### Test result criteria ###
#### Test pass criteria ####
Test case result is a success if the counter option is enabled then "Counters Statistics value is updated into the ovsdb bufmon table". if the counter option is disabled "Counters Statistics value should not update"
#### Test fail criteria ####
Test case result is a fail if counter configuration is enabled "bufmon table counter row is not updating the statistics".


### Test to verify periodic polling interval configuration  ###
#### Description ####
Test to verify that the periodic polling interval changes for buffer monitoring collection.

### Test result criteria ###
#### Test pass criteria ####
Test case result is a success if the collection_period:"5" is changed from 5 to 30 seconds and "Counter statistics updated into the ovsdb bufmon table at every 30 seconds"
#### Test fail criteria ####
Test case result is a fail if the "Counter statistics not updated into the ovsdb bufmon table at every 30 seconds".


### Test to verify trigger based counter collection ###
#### Description ####
Test to verify that the trigger based buffer monitoring counters collection. Set the threshold limit for buffer monitoring counter in the bufmon table and send traffic from the traffic generator.

### Test result criteria ###
#### Test pass criteria ####
Test case result is a success if the threshold limit is crossed "Counter statistics updated in the bufmon table row and status set to triggerd"
#### Test fail criteria ####
Test case result is a fail if the "Counter statistics not updated in the bufmon table row".


### Test cases to verify trigger\_rate\_limit settings ###
#### Description ####
Test cases to verify that the trigger\_rate\_limit for counters collection. Set the threshold limit to lower values and wait for the trigger notification from the switch hardware. Verify the trigger rate limit is applied as per the trigger\_rate\_limit settings.

### Test result criteria ###
#### Test pass criteria ####
Test case result is a success if the trigger rate limit is crossed "Counter statistics not updated in the bufmon table row and trigger notifications from the switch hardware dropped" messages is received.

#### Test fail criteria ####
Test case result is a fail if the "Counter statistics updated in the bufmon table row".
Silk
====

Install SILK. Also usable as a way to play with silk. This role supports a subset of SILK's configuration.

Version
-------

* `1.0.0` --- initial release
* `master` --- latest development version

Requirements
------------

This role is limited to

* CentOS 7

Role Variables
--------------

* `silk_install` --- set to `latest` if you want to upgrade silk, default `installed`
* `silk_conf` --- where to put silk.conf, default `/data/silk.conf`
* `silk_sensor_conf` --- where to put sensor.conf, default `/etc/sysconfig/sensor.conf`
* `silk_sensors:` --- define the sensors
  * `sensor` --- config for a sensor
  * `id` --- the sensor id, should be unique, and not recycled
  * `name` --- name of the sensor
  * `description` --- description of the sensor
  * `type` --- the type of sensor, one of
    * netflow-v5
	* netflow (alias for netflow-v5)
	* ipfix
	* netflow-v9
	* sflow
	* silk
  * `poll_directory` --- where ipfix netflow or silk looks for input. Will be created if it does not exist
  * `listen` --- port to listen at, for netflow-vX, ipfix and sflow
  * `protocol` --- udp or tcp
  * `internal_ipblocks` --- defined in `slik_groups`
  * `external_ipblocks` --- usualy set to `reminder`
* `silk_groups` --- name commonly used definitions
  * `groupdef` --- a group definition
  * `name` --- name of the group definition
  * `ipblocks` --- define blocks of ip addresses
  * `interfaces` --- define interfaces

Example Playbook
----------------

```yaml
---
- hosts: all
  become: true
  roles:
    - role: ../../.
      silk_sensors:
        - sensor:
          id: 0
          name: tcpdump
          description: For importing pcap...
          type: ipfix
          poll_directory: /var/tmp/flows
          internal_ipblocks: "@internal-network"
          external_ipblocks: remainder
        - sensor:
          id: 1
          name: S1
          description: The switch in DMZ
          type: netflow-v9
          listen: 18001
          protocol: udp
          internal_ipblocks: "@internal-network"
          external_ipblocks: remainder

      silk_groups:
        - groupdef:
          name: internal-network
          ipblocks:
            - 10.0.0.0/8 172.16.0.0/12
            - 192.168.0.0/16
        - groupdef:
          name: G01
          interfaces:
            - 1 2 3
            - 4
        - groupdef:
          name: G02
          interfaces:
            - 5 @G01
```

Testing
-------

### Test environment for all OSes

```bash
cd tests
vagrant up
```

Find or create a pcap file, `tcpdump -i eth1 -nn -w testfile.pcap -s 75 not port 22 -c 500`.
Convert the file to a format `rwflowpack` can read, and dump it to the
poll-directory (configured in `/etc/sysconfig/sensor.conf`)

```bash
sudo yaf --silk --noerror --in testfile.pcap --out /var/tmp/flows/testfile.yaf
```

`rwflowpack` is flushing to disk every 120 sec, so wait for 2 min, or see when it is flushing with `sudo journalctl -f`

```bash
[root@localhost vagrant]# rwsiteinfo --fields=sensor,describe-sensor
 Sensor|Sensor-Description|
tcpdump|  for testing only|

[root@localhost vagrant]# rwsiteinfo --sensor tcpdump --fields=type,repo-file-count,repo-start-date,repo-end-date
   Type|File-Count|         Start-Date|           End-Date|
     in|         1|2021/12/16T16:00:00|2021/12/16T16:00:00|
    out|         1|2021/12/16T16:00:00|2021/12/16T16:00:00|
  inweb|         1|2021/12/16T16:00:00|2021/12/16T16:00:00|
 outweb|         1|2021/12/16T16:00:00|2021/12/16T16:00:00|
 innull|         0|                   |                   |
outnull|         0|                   |                   |
int2int|         1|2021/12/16T16:00:00|2021/12/16T16:00:00|
ext2ext|         0|                   |                   |
 inicmp|         0|                   |                   |
outicmp|         0|                   |                   |
  other|         0|                   |                   |

[root@localhost vagrant]# rwfilter --sensors=tcpdump --start=2021/12/16 --proto=0- --type=int2int --pass=test.rw

[root@localhost vagrant]# rwcut test.rw
                                    sIP|                                    dIP|sPort|dPort|pro|   packets|     bytes|   flags|                  sTime| duration|                  eTime| sensor|
                           195.88.55.16|                              10.0.2.15|    0|    0|  1|         2|       168|        |2021/12/16T16:15:55.796|    1.003|2021/12/16T16:15:56.799|tcpdump|
                              10.0.2.15|                           195.88.55.16|    0| 2048|  1|         2|       168|        |2021/12/16T16:15:55.787|    1.002|2021/12/16T16:15:56.789|tcpdump|

```

It is important that the `--start` is the same day as the pcap file was created. Then look at:

* https://tools.netsa.cert.org/silk/docs.html
* https://tools.netsa.cert.org/silk/docs.html#more-analysis

License
-------

GPLv2

Author Information
------------------

Created 2021 by [Sverre Stoltenberg](mailto:sverrest@met.no) for MET Norway.

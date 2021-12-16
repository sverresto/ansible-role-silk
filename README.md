ansible-silk
============

Install silk. Also usable as a way to play with silk.

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

This will change when the config is templated.

* silk_sensor_test
* silk_sensor_config
* silk.conf

Example Playbook
----------------

```yaml
---
- hosts: all
  become: true
  roles:
    - role: ../../.
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

Created 2019 by [Sverre Stoltenberg](mailto:sverrest@met.no) for MET Norway.

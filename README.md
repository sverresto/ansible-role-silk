ansible-silk
============

Install silk.

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

Find or create a pcap file, `tcpdump -i eth1 -nn -w testfile.pcap -s 75`.
Convert the file to a format `rwflowpack` can read, and dump it to the
poll-directory (configured in `/etc/sysconfig/sensor.conf`)

```bash
yaf --silk --noerror -in testfile.pcap --out /var/tmp/flows/testfile.yaf
```

`rwflowpack` is flushing to disk every 120 sec.

```bash
rwfilter --sensors=tcpdump --start=2021/12/16 --proto=0- --type=int2int --pass=test.rw

rwcut test.rw
```

License
-------

GPLv2

Author Information
------------------

Created 2019 by [Sverre Stoltenberg](mailto:sverrest@met.no) for MET Norway.

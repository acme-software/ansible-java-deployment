Ansible Java Deployment
=======================

**Works for all the things packages to a zip / tarball (e.g. sbt native-packager stuff)**

TBD

Tests
-----

By now, there are only manual tests, which basically deploy a test application to different vagrant boxes using the 
deploy role.

### Setup

**Boot vagrant boxes**

```bash
cd test/centos_6
vagrant up
```

**Provide example play app for download:**
```bash
cd test/testapp/play-java-seed
sbt dist
cd target/universal
python -m SimpleHTTPServer
```
**Run test Playbook**

```bash
cd test/playbook
ansible-playbook -i hosts deploy.yml
```

### Tested with
* Ansible 2.4
* Virtualbox 5.2
* Vagrant 2.1 / 1.9.7
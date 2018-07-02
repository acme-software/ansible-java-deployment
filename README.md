Ansible Java Deployment
=======================

**Highly configurable deployment role for (Java/Scala) software packages**

Features
--------

**Downloads and extracts almost any packaged deployment artifact**

Use it to download packages from your Artifactory, Nexus or static package hosting. By now, the role supports e.g.
[SBT Native Packager](https://github.com/sbt/sbt-native-packager) packages, 
[Play Framework 2.x](https://www.playframework.com/) dists and virtually anything runnable packed in a zip or tar(.gz) 
archive.

**Generic application deployment**

The deployment consists of generic actions for downloading, unpacking and moving the application into place, configure 
logging, starting, stoping and restarting the application, and so on. All tasks and templates are highly configurable.

**Service management**

Multiple service runners like `upstart` or `systemd` can be used to register fully functional services for the deployed 
applications. Support for more application daemons is planned.

**Manage multiple application instances on a single machine**

The configurable directory structure, serive management and configuration allows to deploy the same application into 
multiple instances, even on the same server. This can be utilized to achive zero-downtime or A/B deployments.

Usage
-----

As described above, the whole deployment is basically a configurable Ansible role. To use it, download it from GitHub or 
use Ansible Galaxy to install it directly into your existing playbook:

*Galaxy install is not yet ready. See #1 for details.*
```
ansible-galaxy install acmesoftware.java-deployment
```

### Basic Example

If you just want to use it to deploy a simple app instance, there is very little minimal configuration to do. Add the 
following example to your playbook (e.g. `deploy.yml`):


```yml
- hosts: application-servers
  vars:
    deploy_artifact_url: "http://my.artifact.server/app-1.0-SNAPSHOT.zip"
    deploy_app_name: "my-application"
    deploy_service_start_command: "bin/{{ deploy_app_name }} -Dfoo=bar"
  roles:
    - acmesoftware.java-deployment
```

Ideas / Todo
------------

* Use Java Service Wraper (optional)
  * http://yajsw.sourceforge.net/

### Tested with
* Ansible 2.3 / 2.4
* Virtualbox 0.5 / 5.1 / 5.2
* Vagrant 2.1 / 1.9.7

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

### Destination directory structure

On the destination server, the application(s) will be deployed into the `/opt/{{ deploy_app_name }}/` directory. Unless 
configured differently, all directories under that "app root" will be owned by the configured app user and group and 
follow a multi-instance structure. For the basic example above, the following deirectories will be generated:

```
/opt/my-application/
├── downloads
└── instances
    └── 1
        ├── app
        ├── conf
        └── logs
```

The structure above is fully configurable. See [defaults](defaults/main.yml) for config reference.

### Service / Systemloader

This role will, by default, install a service on the target system, which is then used to start & stop each application 
instance. The service wrapper to use can be configured using the `d
eploy_service_type` variable. If not configured, we'll 
chose the default service wrapper for the target system. Set `deploy_service_type: "none"` to do nothing and start the 
deployed app on your own. See [defaults](defaults/main.yml) for config reference.


### Deploy Additional Files

Additional files like an `application.conf`, `logback.xml` or any other file / template can be deployed during the 
process. To do so, see the following config example:

```yml
- hosts: centos6
  vars:
    ...
    deploy_additional_templates:
      - { 
        src: "application.conf.j2", 
        dest: "{{ deploy_dir_config }}/application.conf" 
      }
      - { 
        src: "logback.xml.j2", 
        dest: "{{ deploy_dir_config }}/logback.xml", 
        mode: "0600"
      }
```

```yml
- hosts: centos6
  vars:
    ...
    deploy_additional_copy:
      - { 
        src: "file", 
        dest: "{{ deploy_dir_app }}/file",
        mode: "0644" 
      }
```

Add one item for every file to be deployed and use the following configuration scheme:

| Property  | Mandatory | Description                                   | Default                  |
| --------- | --------- | --------------------------------------------- | ------------------------ |
| src       | yes       | Template to render on local machine           | -                        |
| dest      | yes       | Destination of the file on the local machine  | -                        |
| mode      | no        | Mode (`chmod`) for the destination file       | 0644                     |
| user      | no        | Owner (`chown`) of the destination file       | `{{ deploy_app_user }}`  |
| group     | no        | Owner group (`chown`) of the destination file | `{{ deploy_app_group }}` |

***Note:** These files are deployed per instance, so use the instance directory variables (e.g. 
`{{ deploy_dir_config }}`) in destination path to prevent override.*

All role variables are accessible within those templates, which is especially beneficial when configuring http ports, 
paths, etc. The following example shows a hocon configuration, where those variables are used:

```jinja2
example {
  http {
    # Example to automatically set the application's http port
    # This will output port '9001' for instance 1, '9002' for instance 2 and '9108' for instance 108
    port: 9{{ '%03d'|format(deploy_instance_nr|int) }}

    config-file-ref: {{ deploy_dir_config }}/some_file.txt
  }
}
```

Ideas / Todo
------------

* Use Java Service Wraper (optional)
  * http://yajsw.sourceforge.net/

### Tested with
* Ansible 2.3 / 2.4
* Virtualbox 0.5 / 5.1 / 5.2
* Vagrant 2.1 / 1.9.7
